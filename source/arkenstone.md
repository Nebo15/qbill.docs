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
Our projects are also named in a standardized way.
</aside>

<aside class="notice">
Our ```production```, ```beta``` and ```sandbox``` git branches are protected by a GitHub's "Protected branches" feature. It means that nobody can force-push to this branch; commit directly to it; can merge pull request that didn't passed all CI status checks.

To commit change we are creating pull request and asking a colleague to review it and merge. Nobody is allowed to merge own pull request.
</aside>

## Server Naming

Servers are also consist of following parts:
```{repo_name}.{environment}.{server_id}```, where:

- ```{repo_name}``` - repository name for a deployed project.
- ```{environment}``` - branch/environment name for a deployed project.
- ```{server_id}``` - any human-readable unique id for a server. (Can be picked from a [star common name](https://en.wikipedia.org/wiki/List_of_proper_names_of_stars).)

## DNS Records Naming

(TODO.)

For ```api``` projects: ```{environment}``` subdomain for a ```beta``` and ```sandbox``` environment. ```api``` subdomain for ```production`` environment.

For ```web``` projects: ```{environment}-web``` subdomain for a ```beta``` and ```sandbox``` environment. Root domain for ```production``` environment. (And a CHNAME ```www``` to ```@```.)

<aside class="notice">
We don't use 3-d level subdomains because it's would be too expensive to generate valid SSL wildcard certificates for all of them.
</aside>


## Single Responsibility

(TODO: Single VM for all envs? If else, droplet naming doesn't have sense).

For production environment we are sticking to "single VM responsibility" strategy. It means that we do not deploy more than one project to a single DO droplet. It helps to scale better and simplifies maintenance of our projects.

For development environment we can have multiple projects on one VM, but still we are trying to avoid it. VM's are cheaper than cost of developer work.

# SSH Authorization

1. Central server
2. Generate user on it and generate public/private key on a central server.
3. Provision public key from central server to application servers.
4. Allow SSH access to application servers only from central server.
5. Provision Areknstone password changes to all application servers. Provision SSH public key changes to a central server.

# GitFlow

# Security

Never commit passwords, private keys, API keys or anything like to a git repos. If you did something like this, than you must change all credentials that was affected.


# Continuous Integration

## Testing

## Delivery

# Deploy Strategy

1. Создать сервер на DO и задеплоить туда Rome с базовыми Puppet правилами:
- Puppet Apply базовых настроек rome
- HTTP сервис для управления инстансом по API
- SSH (порт и мастер-ключ, IP с которого можно авторизоваться)
- Автоапдейт
- Диффи-Хелман (прод)
..


2. Деплой проекта (или изменений проекта) на сервер:
- Стянуть git
- Стянуть puppet модули
- Puppet apply настроек с git репозитория
- Собрать проект в releases/:commit_id
- Запустить releases/:commit_id/bin/build.sh
- Сделать симлинк current -> releases/:commit_id-:commit_time

(Все конфиги и публичные SSH ключи лежат прямо в репозитории проекта. Разделены по папкам. Роль берется с суффикса названия сервера, например mbank.api.production.s838 -> mbank.api::production::s838)

2.1. Взять все папки в puppet/environments/:env/users и создать для них юзеров с указанным публичным ключем, но без пароля (для запуска SUDO человек должен сначала должен создать себе пароль через ssh). Заставить человека менять пароль при первом входе.


3. CRUD SSH ключей с доступом на сервер
API для создания пользователей и добавления туда публичных ключей


4. Интеграция со Slack (уведомления и деплой через команду) и https://developer.github.com/v3/repos/deployments/

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
