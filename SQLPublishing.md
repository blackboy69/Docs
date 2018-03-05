# SQL publishing

## Problem
SQL management is a manual process. This causes environments to get out of sync, and creates lots of busy work for the QA team. When an environment is no longer working due to missing SP fields, columns or tables, developers, QA and business analysts become blocked. The fix requires much research and trial and error to fix.

Production sql publishes are also manual by hand. Production sql updates pushes are only successfull due  **QA diligence**.

Since sql is  manual, testing the sql scripts is a hit or miss affair. If a developer fails to attach/update thier sql script, chaos ensues. The QA team must then track down the correct sql using out of band email slack etc. 

All this makes both publishing and code pushes to environments difficult and error prone. 

The goal is to make sql publising is repeatable and automated.

## Solution
### SQL Publishing Console UI
A Sql publishing console would used to publish SQL code to databases.
The UI is very simple as follows;

|||
|---|---
|Select Jira Issues| [OLP-12,OLP-321,EN-32,EN-553 ] |
|Choose Branch| [List-Of-Branches-From-Plastic] |
|Choose Database| [List-of-Onelegal-Databases]
|**RUN SQL**|**ROLLBACK SQL**|
_________________________

Console UI will also have a "configuration" page that handles configuring Plactic endpoints/credentials & database connection strings.

## Phase 1: Commit developer sql scripts to source control
Create a new folder structure in source control called "SQL". This will contain all schema and data modification scripts.

```
SQL\Schema
SQL\Scripts
```

Changes to sql schema must go in the "schema" folder, changes to sql scripts go into the "Scripts" folder. This is seperated to allow for schema changes to run before data loading/modification scripts.

*File naming*
Files will be named after the issue number. All scripts shall have a rollback script assocated with them. Rollback script should have a `_rollback` appeneded to the name.

```
SQL\Schema\OLP-1235.sql
SQL\Schema\OLP-1235_rollback.sql

SQL\Scripts\OLP-1235.sql
SQL\Scripts\OLP-1235_rollback.sql
```

**Caveats**

- SQL scripts must allow multiple runs without side effects
- The SQL publish shall fail if there is no rollback script present. 

## Phase 2: Use EF to generate SQL scripts
This step uses the Entity framework to generate sql scripts so developers doe not need to do it manually. Developers would modify models, then run a script to generate the sql based model changes. Entity framework supports this via the built in `enable-migration`, `add-migration`, `update-database` workflow. 

This explicitly **does not** use the built in migrations to update the database. The built in migrations are too dangerous to be enabled in a live production application. Sql publishing would still be manual. 

The script will be automatically generated and checked into source control by the develoer.

**Phase 2 needs more refinement** but the basic Idea is to generate sql instead of writing it by hand. 





