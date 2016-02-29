---
title: Project Arkenstone

language_tabs:
  - shell

toc_footers:
 - <a href='index.html'>Billing API</a>
 - <a href='index.html'>Applications API</a>

search: true
---

# Introduction

This is project that Nebo15 uses to manage it's infrastructure.

We use:

- [DigitalOcean](https://digitalocean.com) (DO) as our main hosting provider, DNS hosting and Floating IP provider.
- DO [CloudConfig](https://www.digitalocean.com/community/tutorials/an-introduction-to-cloud-config-scripting) for initial VM (Droplet) configuration.
- [Puppet](https://puppetlabs.com/puppet/puppet-open-source) for configuration management.
- [GitHub](https://github.com/) as VCS.
- [NewRelic](https://newrelic.com) for server monitoring.
- [Travis-CI](https://travis-ci.com) as a CI and build server.

Why DigitalOcean? Because it's cheap, friendly and reliable.

# Infrastructure overview

## Repository and Project Naming

Everybody must name new projects in a agreed way, so we can make assumptions about project, server purpose and deployed environment based on a single string.

Generally repository name should look something like this:
```{project_name}[.{subproject_name}].{project_type}``` => ```{repo_name}```, where:

- ```{project_name}``` - lowercased alphanumeric string, internal name for a project. Usually picked from Star Trek, Lord of The Rings or Star Wars movies, with some kind of associative connection.
- ```{subproject_name}``` - (optional) lowercased alphanumeric string, subproject name.
- ```{project_type}``` - lowercased alphanumeric string, repo type.

There are 5 known repo types:

- ```api``` - REST API back-end.
- ```web``` - HTML5+CSS3 and Angular.js thin client.
- ```layout``` - HTML5 layout for corresponding ```web``` project.
- ```ios``` - iOS application.
- ```android``` - Android application.
- ```docs``` - project documentation. (TODO: Keep it in same repo?)

In our GitHub account only forked repos can be named in a different way. For Composer-included repos we replace dots (```.```) with dashes (```-```).

```
mbank.api
mbank.web
mbank.layout
mbank.emails.layout
mbank.persona.api
mbank.webclient.web
mbank.ios
mbank.android
```

## Environments

For all projects we have 3 environments:

- ```production``` - stable environment with at least 99% SLA.
- ```beta``` - pre-production environment, where we can test change before they merged into ```production```. We can skit it for very small projects without external API dependencies and where we can run all the tests on ```sandbox```.
- ```sandbox``` - development environment and environment for our API consumers, whenever they need sandbox to test their code.

## Branch Naming

All our environments are mapped into git branches. It helps us to keep in mind where code will be provisioned whenever we add a new commit. And helps to get rid of messy branch->environment mapping.

<aside class="notice">
Our ```production```, ```beta``` and ```sandbox``` git branches are protected by a GitHub's "Protected branches" feature. It means that nobody can force-push to this branch; commit directly to it; can merge pull request that didn't passed all CI status checks.

To commit change we are creating pull request and asking a colleague to review it and merge. Nobody is allowed to merge own pull request.
</aside>

## Server Naming

Servers are named based on their hostname and thus a DNS host name. This helps us to avoid PTR records issues.

Initially we name them in the following way:
```{server_id}.nebo15.com```, where:

- ```{server_id}``` - any human-readable unique id for a server. (Can be picked from a [star common name](https://en.wikipedia.org/wiki/List_of_proper_names_of_stars).)
- ```{environment}``` - branch/environment name for a deployed project.

## DNS Records Naming

When we are adding a new server to our pool, we automatically adding a new DNS record to our ```nebo15.com``` domain. If you need to map another subdomain or domain to the server, you can simply setup a ```CNAME``` DNS record and make corresponding changes in nginx settings.

```
gandalf.forza.md CNAME gandalf.nebo15.com
```

Generally we have a convention on naming end-domain endpoints:

- For ```api``` projects: ```{environment}``` subdomain for a ```beta``` and ```sandbox``` environment. ```api``` subdomain for ```production`` environment.
- For ```web``` projects: root domain for ```production``` environment. (And a redirect from ```www``` to ```@```.) Other environments are tested on ```nebo15.com``` domain.

<aside class="notice">
We don't use 3-d level subdomains because it's would be too expensive to generate valid SSL wildcard certificates for all of them.
</aside>

## Single Responsibility

For production environment we are sticking to "single VM responsibility" strategy. It means that we do not deploy more than one project to a single DO droplet. It helps to scale better and simplifies maintenance of our projects.

For development environment we can have multiple projects on one VM, but still we are trying to avoid it. VM's are cheaper than cost of developer work.

# SSH Authorization

All SSH access should be done trough middleware Arkenstone Auth server. It means that to connect, for example, to ```example.nebo15.com```, you need to run:

- ```$ ssh auth.nebo15.com -p2020```.
- ```# ssh example.nebo15.com -p2020```.

Arkenstone will automatically:

- Generate a SSH keypair for you and provision it to all servers you have access to.
- Provision your Arkenstone password as to all your accounts on all servers you have access to.
- Update ```~/.ssh/authorized_keys``` when you change ```public-key``` in Arkenstone GUI.

# Security

Never commit passwords, private keys, API keys or anything like to a git repos. If you did something like this, than you must change all credentials that was affected.

# Continuous Integration

## Testing

All our applications have minimum code coverage requirement (75%). On each commit we will trigger a build and test on Travis-CI server, that will run following checks:

- Code style.
- Code coverage.
- All project tests.

If one of checks is failed, than you should not merge your feature branch into repo. (And, actually, you won't be able to.)

## Delivery

# Initialization

Arkenstone will create a DigitalOcean droplet with specified parameters, using last DO Ubuntu LTS image. Only key that is added to server is a Arkenstone master key.

After that it will set a CloudConfig to this VM, that will init Puppet base configuration on this server:

- Change ssh port and ssh authorization settings (port: 2020; deny root login; authorization with key-only; allow login only from Arkenstone master-server IP address).
- Setup nginx.
- Setup Arkenstone client app and make it available only from Arkenstone Master-Server IP address.
- Deploy Elliptic-Curve Diffie-Hellman (ECDHE) group.
- Some other basic configurations.

# Deploying a Project

1. Notify Slack that deployment is started.
1. Asks you to authorize our Arkenstone GitHub app.
2. Generate a SSH key pair for GitHub repo on a application-server.
3. Add this key to a GitHub Deploy Keys.
4. Setup a virtual host to access GitHub that uses this keys from this server (in ```~/.ssh/config```).
3. Fetch project git repository to a ```/www/{project_name}/releases/latest``` folder.
4. Change branch to a deployment environment.
5. Make a project copy (without ```.git``` folder) to a ```/www/{project_name}/releases/{commit_id}-{deploy_timestamp}/``` and ```cd``` to it.
6. Provision users ssh keys and passwords from a Arkenstone master-servers.
7. Fetch all Puppet dependencies and apply project config.
8. Apply a Puppet config for this project.
9. Fetch all Puppet dependencies.
10. Run ```/www/{project_name}/releases/{commit_id}-{deploy_timestamp}/bin/build.sh```.
11. Create symlink from ```/www/{project_name}/releases/{commit_id}-{deploy_timestamp}/``` to a ```/www/{project_name}/releases/current```.
11. Notify NewRelic, [GitHub](https://developer.github.com/v3/repos/deployments/) and Slack that deployment is finished.
12. Add a project server to it's load balancer.

All deploy logs are stored in ```/var/logs/www/{project_name}/releases/{commit_id}-{deploy_timestamp}/deploy.log```.

<aside class="notice">
We store all project configurations inside a repo along with a source code. So you can change add components in one place. And, eventually, all people that have access to the source code, can add themselves to a project users.

Puppet config is in "/arkenstone/{environment}/puppet/init.pp".
Users that have access is listed in "/arkenstone/{environment}/users.json".
Environment variables listed in "/arkenstone/{environment}/environment.json".
Load balancer discovery settings listed in "/arkenstone/{environment}/lb.json".
</aside>

# Deploying an update

Deployment for a changes are very familiar. Just re-run steps 3-10 from previous section.

# Service Discovery and Load Balancing

Each time you deploy a application we will try to find a appropriate load balancer for it and add a new node to it's config. It helps to scale projects really fast.

# Users

All Arkenstone users and their changes are automatically provisioned to a main SSH authorization server and to all application servers that user have access to.

## List all active Users

```
GET /users
{
  users: [
    {name: "andrew", public_key: "ssh-rsa lsslkjsjlsldl", password_hash: "sdsdsdsdsd", email:"email@examile.com"}
  ]
}
```

## Create a User

User name is unique string. We are stripping email from public key, to avoid Linux key conflicts.

```
POST /users
{
  name: "andrew",
  public_key: "ssh-rsa lsslkjsjlsldl",
  password_hash: "sdsdsdsdsd",
  email:"email@examile.com"
}
```

## Get a User

```
GET /users/:name
{
  name: "andrew",
  public_key: "ssh-rsa lsslkjsjlsldl",
  password_hash: "sdsdsdsdsd",
  email:"email@examile.com"
}
```

## Update a User

```
PUT /users/:name
{
  name: "andrew",
  public_key: "ssh-rsa lsslkjsjlsldl",
  password_hash: "sdsdsdsdsd",
  email:"email@examile.com"
}
```


## Delete a User

```
DELETE /users/:name
```

# Application Servers

## List all Application Servers

```
GET /servers
{
  deployments: [
    {
      repo: ":repo_name",
      environment: ":environment",
      deployments: [
        {commit_id: "", time: "", user: ":user_id", is_current: true}
      ]
    }
  ],
  ssh: {
    users: [:user_id, :user_id, :user_id]
  },
  droplet: {
    region: ":do_region",
    size: ":do_size",
    ipv6: true,
    private_networking: true,
    ip: "",
    name: "",
    private_ip: ""
  }
}
```

## Create and provision a new Application Server

```
POST /servers
{
  deployments: [
    {
      repo: ":repo_name",
      environment: ":environment"
    }
  ],
  ssh: {
    users: [:user_id, :user_id, :user_id]
  },
  droplet: {
    region: ":do_region",
    size: ":do_size",
    ipv6: true,
    private_networking: true
  }
}
```

## Provision changes to a Application Server

```
PUT /servers/:id
{
  deployments: [
    {
      repo: ":repo_name",
      environment: ":environment"
    }
  ],
  ssh: {
    users: [
      :user_id
    ]
  }
}
```

## Halt an Application Server

```
DELETE /servers/:id
```

## Stop an Application Server

```
HALT /servers/:id
```

# Deployments

## List all Deployments for a Server

```
GET /servers/:id/deployments
{
  deployments: [
    {
      repo: ":repo_name",
      environment: ":environment",
      deployments: [
        {commit_id: "", time: "", user: ":user_id", is_current: true}
      ]
    }
  ]
}
```

## Get Project deployments list for an Application Server

```
GET /servers/:id/deployments/:repo_name
{
  repo: ":repo_name",
  environment: ":environment",
  deployments: [
    {commit_id: "", time: "", user: ":user_id", is_current: true}
  ]
}
```

## Get Project deployment data for an Application Server

```
GET /servers/:id/deployments/:repo_name/:commit_id
{
  commit_id: "",
  time: "",
  user: ":user_id",
  log: "",
  is_current: true
}
```

## Deploy new version of Application

```
POST GET /servers/:id/deployments/:repo_name
{
  commit_id: ":commit_id",
  is_current: true
}
```

## Rollback to a previous version of Application

Also this allows to deploy specific version of application to a server.

```
PUT /servers/:id/deployments/:repo_name/:commit_id
{
  is_current: true
}
```

## Get interactive Deployment log

```
GET /servers/:id/deployments/:repo_name/:commit_id/log
```

# Webhooks

POST /webhooks/github/
POST /webhooks/slack/
POST /webhooks/newrelic/

# TODOs

Refs:

- [StrongLoop Similar Service](https://strongloop.com/node-js/build-deploy-and-scale/).
- Arkenstone client API.
