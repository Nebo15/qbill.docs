---
title: Project Gandalf

language_tabs:
  - shell

toc_footers:
 - <a href='index.html'>Accounting API - Mirian</a>
 - <a href='app.html'>Projects API - Mirian SaaS</a>
 - <a href='arkenstone.html'>Deployment API - Arkenstone</a>
 - <a href='palantir.html'>API Service Bus - Palantir</a>
 - <a href='gandalf.html'>Risk Management - Gandalf</a>

search: true
---

# Introduction

Gandalf is a Open-Source Decision Engine for Big-Data.

Source code is available in our [GitHub Account](https://github.com/Nebo15/gandalf.api/). It's split in two separate parts:

- [PHP, Laravel, MongoDB Back-End](https://github.com/Nebo15/gandalf.api/)
- [Angular.js GUI](https://github.com/Nebo15/gandalf.web/)

### Features

- Customizable - have the freedom to design a decision rules you need and to manage data structure on the fly in a easy to understand way.
- Split testing (aka. Champion Winner) - each table can have infinite number of different variants, you can allocate traffic between them and leave only best one later.
- Decision History - all decisions are saved with all metadata: request parameters, decision table at request moment of time and decision itself.
- Decision Analytics - you can review all decisions made by your tables and to analyze what rules is triggered more often.
- Revisions History - rollback any changes that was made by you or your collaborators.
- Tooling - debug your decision tables directly from Gandalf GUI.
- SASS-based Role Model and oAuth 2.0 - you can create projects and share access among users to them.
- Production-tested - several large NDA-closed PSP's and online lending platforms already use Gandalf.

# Use Cases

## Decision Table and Rules Engine

There are many cases when you need to make decision based on input data. One good example is a decision to approve or decline lending application. You can setup a set of cut-off risk rules to decline applications for a high-risk applicants or to use a decision table to specify which categories of users will receive a loan on automatic basis.

## Scoring Table

Instead of rule decision you can set a score point that will be added to a final result.
In a result you will get a total of all scores and you can make further decision based on this value.

## State Transition Rules for BPM

Sometimes you need to have very complex rules in your BPM-engine, that should tell you to which branch your task should be transitioned to. Use Gandalf to manage all those rules.

# Support

This project is continuously supported on behalf of our customers and as part of our own [SaaS-service](https://gandalf.nebo15.com/). We recommend to use it, you can create your own installation of Gandalf.

Feel free to submit tickets in our GitHub repos, we will respond shortly. And if you want some custom conditions, e.g. 99.9% SLA - [contact us](mailto:support@nebo15.com).

# How does it work?

You simply create a decision or scoring table, define some rules and query this table via API.

It makes Gandalf suitable for anti-fraud, loan scoring, transaction scoring, risk management, and as part any other decision making processes.

# Tables

Decision table is the main entity in Gandalf. It consists of columns that describe API request structure, rows that represent decision-making logic, and cells which describe a single validation rule.

### Decision Table

Decision table will apply all rules one by one, until one rule is passed and return value in Decision column for this rule.

### Scoring Table

Scoring table will sum all values in Scoring column and return their total.

## Columns (Fields)

Column represent specific request parameter. You can add additional parameters on a fly by adding a column. After saving table with new column all future API request should provide data for it. (Or at least ```null```, if you want to sometimes omit some request parts.)

<aside class="notice">
All request parameters are required!
</aside>

However, API consumer can provide more data and you don't need to specify column for each of request fields, in this case we will skip unused ones. Best practice is to feed table with any data you have, so you can use it when you need without changing back-end.

Column can have following settings:

- ```Title``` - human-readable string that describes field in interface.
- ```Field API Key``` - field name from witch data will be taken. For example, if you add a field with key ```name``` than all API consumers should provide JSON with this field: ```{"name": "Some User Name"}```.
- ```Type``` - type of data that will be submitted in this field. ```String```, ```Numeric``` or ```Boolean```.

### Presets

You can modify field value for table rows by adding field preset. For example, you have field called salary and it's too routine to add a "salaries greater than 1000" condition in each row, instead you can create preset that turns ```Numeric``` salary into ```Boolean``` type and simply turn on this validation in each rule.

It should look something like this:

- ```Title``` = ```Sufficient Salary```;
- ```Field API Key``` = ```salary```;
- ```Type``` = ```Numeric```;
- ```Preset Condition``` = ```greater than```;
- ```Preset Value``` = ```1000```.

By checking checkbox below ```Low Salary```  column in a row, you will make sure that this row won't pass check untill ```salary``` is greater than ```1000```.

## Rows (Rules)

Row represents a single rule in decision table. All rows are checked in order it was defined. (You can reorder it by drag'n'drop.) Basically, you can think about relation between rows as ```OR``` logical operators (or as ```else if```'s).

Row will return value selected in "Decision" column only if all validation rules in it have passed. Rules are checked in a same order as you see them in a table. You can reorder them by drug'n'drop.

If no one of defined rows passed all conditions, then we will return ```final_decision``` that equals a value specified in "Default Decision" dropdown.

> ```
IF rule1 then return rule1.decision
ELSEIF rule2 then return rule2.decision
ELSEIF rule3 then return rule3.decision
ELSEIF ruleN then return ruleN.decision
ELSE default return default.decision
```

## Cells (Conditions)

All cells in a row represent validations in an ```AND``` logical operator style (or you can thing about them as a bunch of conditions joined with ```&&``` inside ```if``` statement).

Sometimes you have a big table and in some rows you prefer to skip some validations. For this case you can select special validation rule called ```is set```. Logically it means that ```{field_name} is set``` and this condition will always pass validation.

> ```
IF rule1(cellCondition1 && cellCondition2 && cellConditionN) then return rule1.decision
ELSEIF ruleN(cellCondition1 && cellCondition2 && cellConditionN) then return ruleN.decision
ELSE default return default.decision
```

### Available Conditions

Available rules can differ based on a column type. Generally you should consider all rules logic as follows:
```{request_field_vale} {condition} {condition_value}```. For some conditions you can omit their value.

String fields support following conditions:

- ```=``` - validation will pass if field value equals specified value.
- ```!=``` - validation will pass if field value does not equal specified value.
- ```in``` - validation will pass if field value eqals to one of listed values. Separate values by comma with space. If searched string have comma you can surround value by single qoute. For example: ```d,e``` in ```a, b, c, 'd,e'``` will return true.
- ```not in``` - validation will pass if field value does not eqal to any of listed values.
- ```contains``` - validation will pass if field value is contains specified value.
- ```is set``` - validation will always pass. (Use it to skip some columns.)
- ```is null``` - validation will pass if field value equals ```null```. (Use it check if some column is skipped.)

Numeric supports:

- ```=``` - validation will pass if field value equals specified value.
- ```>``` - validation will pass if field value is greater than specified value.
- ```>=``` - validation will pass if field value is greater or equal to a specified value.
- ```<``` - validation will pass if field value is less than specified value.
- ```<=``` - validation will pass if field value is less or equal to a specified value.
- ```!=``` - validation will pass if field value does not equal specified value.
- ```in``` - validation will pass if field value eqals to one of listed values. Separate values by comma with space. If searched string have comma you can surround value by single qoute. For example: ```d,e``` in ```a, b, c, 'd,e'``` will return true.
- ```not in``` - validation will pass if field value does not eqal to any of listed values.
- ```is set``` - validation will always pass. (Use it to skip some columns.)
- ```is null``` - validation will pass if field value equals ```null```. (Use it check if some column is skipped.)

Boolean supports:

- ```true``` - will pass if field value is ```true```, ```1```, ```"1"``` or ```'1'```.
- ```false``` - will pass if field value is ```false```, ```0```, ```"0"``` or ```'0'```.
- ```is set``` - validation will always pass. (Use it to skip some columns.)
- ```is null``` - validation will pass if field value equals ```null```. (Use it check if some column is skipped.)

## Revisions

All changes that is made to a table is saved in Revisions history. You can see who changed what, and rollback to any previous state of the table.

Rollback will create a new Revision, so it can be canceled.

## Split Testing

Each table can have multiple variants. By adding a Variant you will start a Split Test for a table.

All variants share same fields and this request structure, but can have different set of rules. It allows to test your hypothesis on how to make decision better, to measure improvement in third-party tool and to continuously improve your decision flow.

You can manage traffic allocation for all variants, for example sending only 5% of requests to variant B.

## Decision Analytics

Each time Decision Table is called we will calculate how many times single condition was true, and how many times each rule returned it's decision. It allows you to see what rules work in real life, and to find ways to optimize your table.

# Decision History

All successful API request will lead to creating a Decision. You can find all Decisions in "Hisotry" tab.

Decision shows all request parameters, decision engine response and decision table that was applied when request was made. It means that whenever you change your decision table, it won't affect history and you still will be able to understand decision flow for each moment of time.

You can retry request on a latest table version by clicking "Debug" in request well.


# Installation Guide

## Back-End

You can use [Vagrant](https://www.vagrantup.com/) to intialize a development environment. Simply install it, ```cd``` into your project directory and run:

```
vagrant up
```

Also you can notice that we use [Puppet](https://puppetlabs.com/puppet/puppet-open-source) as our configuration tool. Scripts located in [/puppet](https://github.com/Nebo15/gandalf.api/tree/master/puppet) directory of this repo. You can deploy code to your server and simply run:

```
set -o nounset -o errexit -o pipefail -o errtrace
trap 'error "${BASH_SOURCE}" "${LINENO}"' ERR
echo 127.0.0.1 gandalf.yourdomain.com | sudo tee -a /etc/hosts
sudo /bin/bash puppet/initial/init.sh -u "www-data"
```

And your server will be ready for production use!

## Front-End

# API Docs

Right now API docs can be found [here](http://gandalf.nebo15.com/doc/?url=https://raw.githubusercontent.com/Nebo15/gandalf.api/develop/public/swagger.json#!/User/get_users). Also you can find [SWAGGER file](https://github.com/Nebo15/gandalf.api/blob/develop/public/swagger.json) in our API repo.

# Story behind Gandalf

We - are production oriented team, and we work for a fintech company. Almost all project that we created need a decision engine that reduces business risks. And all solutions we can find is ether old and ugly, or very expensive. So we decided to create free open-source alternative, that will be scalable, reliable and flexible enough to cover 95% of cases.

To make is suitable for Big Data we decided to build it on top of use very reliable open-source database MongoDB, that have good sharding capabilities and easy to maintain.

Also we believe that vendor-lock is a bad thing, so we published all source code on a MIT license, so you are free to change it as you wish.

Done by [Nebo #15](http://nebo15.com/).
