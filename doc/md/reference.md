---
title: CLI Reference
id: cli-reference
slug: cli-reference
---
<!-- This file is generated from cmd/atlas/doc.tmpl, do not edit manually -->
## Introduction

This document serves as reference documentation for all available commands in the Atlas CLI.
Similar information can be obtained by running any atlas command with the `-h` or `--help`
flags.

For a more detailed introduction to the CLI capabilities, head over to the
[Getting Started](/getting-started/) page.

## Supported Version Policy

To ensure the best performance, security and compatibility with the Atlas Cloud service, the Atlas team
will only support the two most recent minor versions of the CLI. For example, if the latest version is
`v0.25`, the supported versions will be `v0.24` and `v0.25` (in addition to any patch releases and the
"canary" release which is built twice a day).

## Distributed Binaries

:::info Old Versions

As part of our **Supported Version Policy** mentioned above, binaries for versions that were published
more than 6 months ago will be removed from the CDN and Docker Hub.

:::

The binaries and Docker images distributed in official releases are released in two flavors:

* Atlas - A binary built from the OSS repository in addition to proprietary code maintained by
  [Ariga](https://ariga.io), including access to the Atlas Cloud service and commercial driver support.
  Atlas is distributed under the [Atlas EULA](https://ariga.io/legal/atlas/eula).
* Atlas Community - A binary built from the OSS repository only. Atlas Community is distributed under
  the [Apache 2 License](https://github.com/ariga/atlas/blob/master/LICENSE).

For instructions on how to install Atlas Community, see this [guide](/community-edition).


## atlas license

Display license information

#### Usage
```
atlas license
```


## atlas login

Log in to Atlas Cloud.

#### Usage
```
atlas login [org] [flags]
```

#### Details
'atlas login' authenticates the CLI against Atlas Cloud.

#### Flags
```
      --token string   API token to authenticate to an Atlas Cloud organization

```


## atlas logout

Logout from Atlas Cloud.

#### Usage
```
atlas logout
```

#### Details
'atlas logout' removes locally-stored credentials.


## atlas migrate

Manage versioned migration files

#### Usage
```
atlas migrate
```

#### Details
'atlas migrate' wraps several sub-commands for migration management.

#### Flags
```
  -c, --config string        select config (project) file using URL format (default "file://atlas.hcl")
      --env string           set which env from the config file to use
      --var <name>=<value>   input variables (default [])

```


### atlas migrate apply

Applies pending migration files on the connected database.

#### Usage
```
atlas migrate apply [flags] [amount]
```

#### Details
'atlas migrate apply' reads the migration state of the connected database and computes what migrations are pending.
It then attempts to apply the pending migration files in the correct order onto the database. 
The first argument denotes the maximum number of migration files to apply.
As a safety measure 'atlas migrate apply' will abort with an error, if:
  - the migration directory is not in sync with the 'atlas.sum' file
  - the migration and database history do not match each other

If run with the "--dry-run" flag, atlas will not execute any SQL.

#### Example

```
  atlas migrate apply -u "mysql://user:pass@localhost:3306/dbname"
  atlas migrate apply --dir "file:///path/to/migration/directory" --url "mysql://user:pass@localhost:3306/dbname" 1
  atlas migrate apply --env dev 1
  atlas migrate apply --dry-run --env dev 1
```
#### Flags
```
  -u, --url string                [driver://username:password@address/dbname?param=value] select a resource using the URL format
      --dir string                select migration directory using URL format (default "file://migrations")
      --format string             Go template to use to format the output
      --revisions-schema string   name of the schema the revisions table resides in
      --dry-run                   print SQL without executing it
      --lock-timeout duration     set how long to wait for the database lock (default 10s)
      --baseline string           start the first migration after the given baseline version
      --tx-mode string            set transaction mode [none, file, all] (default "file")
      --exec-order string         set file execution order [linear, linear-skip, non-linear] (default "linear")
      --allow-dirty               allow start working on a non-clean database

```


### atlas migrate checkpoint

Generate a checkpoint file representing the state of the migration directory.

#### Usage
```
atlas migrate checkpoint [flags] [tag]
```

#### Details
The 'atlas migrate checkpoint' command uses the dev-database to calculate the current state of the migration directory
by executing its files. It then creates a checkpoint file that represents this state, enabling new environments to bypass
previous files and immediately skip to this checkpoint when executing the 'atlas migrate apply' command.

#### Example

```
  atlas migrate checkpoint --dev-url docker://mysql/8/dev
  atlas migrate checkpoint --dev-url "docker://postgres/15/dev?search_path=public"
  atlas migrate checkpoint --dev-url "sqlite://dev?mode=memory"
  atlas migrate checkpoint --env dev --format '{{ sql . "  " }}'
```
#### Flags
```
      --dev-url string          [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --dir string              select migration directory using URL format (default "file://migrations")
      --dir-format string       select migration file format (default "atlas")
  -s, --schema strings          set schema names
      --lock-timeout duration   set how long to wait for the database lock (default 10s)
      --format string           Go template to use to format the output
      --qualifier string        qualify tables with custom qualifier when working on a single schema
      --edit                    edit the generated migration file(s)

```


### atlas migrate diff

Compute the diff between the migration directory and a desired state and create a new migration file.

#### Usage
```
atlas migrate diff [flags] [name]
```

#### Details
The 'atlas migrate diff' command uses the dev-database to calculate the current state of the migration directory
by executing its files. It then compares its state to the desired state and create a new migration file containing
SQL statements for moving from the current to the desired state. The desired state can be another another database,
an HCL, SQL, or ORM schema. See: https://atlasgo.io/versioned/diff

#### Example

```
  atlas migrate diff --dev-url "docker://mysql/8/dev" --to "file://schema.hcl"
  atlas migrate diff --dev-url "docker://postgres/15/dev?search_path=public" --to "file://atlas.hcl" add_users_table
  atlas migrate diff --dev-url "mysql://user:pass@localhost:3306/dev" --to "mysql://user:pass@localhost:3306/dbname"
  atlas migrate diff --env dev --format '{{ sql . "  " }}'
```
#### Flags
```
      --to strings              [driver://username:password@address/dbname?param=value] select a desired state using the URL format
      --dev-url string          [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --dir string              select migration directory using URL format (default "file://migrations")
      --dir-format string       select migration file format (default "atlas")
  -s, --schema strings          set schema names
      --lock-timeout duration   set how long to wait for the database lock (default 10s)
      --format string           Go template to use to format the output
      --qualifier string        qualify tables with custom qualifier when working on a single schema
      --edit                    edit the generated migration file(s)

```


### atlas migrate down

Reverting applied migration files from the database

#### Usage
```
atlas migrate down [flags] [amount]
```

#### Example

```
  atlas migrate down -u "mysql://user:pass@localhost:3306/dbname"
  atlas migrate down --env prod --to-version 20230102150405
  atlas migrate down --env prod --to-tag e29be4e
```
#### Flags
```
  -u, --url string                [driver://username:password@address/dbname?param=value] select a resource using the URL format
      --to-version string         desired version to revert to
      --to-tag string             desired tag to revert to
      --dir string                select migration directory using URL format (default "file://migrations")
      --dev-url string            [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --format string             Go template to use to format the output
      --dry-run                   print SQL without executing it
      --revisions-schema string   name of the schema the revisions table resides in
      --lock-timeout duration     set how long to wait for the database lock (default 10s)

```


### atlas migrate edit

Edit a migration file by its name or version and update the atlas.sum file.

#### Usage
```
atlas migrate edit [flags] {name | version}
```

#### Example

```
  atlas migrate edit 20060102150405
  atlas migrate edit 20060102150405 --env dev
  atlas migrate edit 20060102150405_name.sql --dir file://migrations
```
#### Flags
```
      --dir string          select migration directory using URL format (default "file://migrations")
      --dir-format string   select migration file format (default "atlas")

```


### atlas migrate hash

Hash (re-)creates an integrity hash file for the migration directory.

#### Usage
```
atlas migrate hash [flags]
```

#### Details
'atlas migrate hash' computes the integrity hash sum of the migration directory and stores it in the atlas.sum file.
This command should be used whenever a manual change in the migration directory was made.

#### Example

```
  atlas migrate hash
```
#### Flags
```
      --dir string          select migration directory using URL format (default "file://migrations")
      --dir-format string   select migration file format (default "atlas")

```


### atlas migrate import

Import a migration directory from another migration management tool to the Atlas format.

#### Usage
```
atlas migrate import [flags]
```

#### Example

```
  atlas migrate import --from "file:///path/to/source/directory?format=liquibase" --to "file:///path/to/migration/directory"
```
#### Flags
```
      --from string         select migration directory using URL format (default "file://migrations")
      --to string           select migration directory using URL format (default "file://migrations")
      --dir-format string   select migration file format (default "atlas")

```


### atlas migrate lint

Run analysis on the migration directory

#### Usage
```
atlas migrate lint [flags]
```

#### Example

```
  atlas migrate lint --env dev
  atlas migrate lint --dir "file:///path/to/migrations" --dev-url "docker://mysql/8/dev" --latest 1
  atlas migrate lint --dir "file:///path/to/migrations" --dev-url "mysql://root:pass@localhost:3306" --git-base master
  atlas migrate lint --dir "file:///path/to/migrations" --dev-url "mysql://root:pass@localhost:3306" --format '{{ json .Files }}'
```
#### Flags
```
      --dev-url string      [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --dir string          select migration directory using URL format (default "file://migrations")
      --dir-format string   select migration file format (default "atlas")
      --format string       Go template to use to format the output
      --latest uint         run analysis on the latest N migration files
      --git-base string     run analysis against the base Git branch
      --git-dir string      path to the repository working directory (default ".")
  -w, --web                 open the lint report in the browser

```


### atlas migrate new

Creates a new empty migration file in the migration directory.

#### Usage
```
atlas migrate new [flags] [name]
```

#### Details
'atlas migrate new' creates a new migration according to the configured formatter without any statements in it.

#### Example

```
  atlas migrate new my-new-migration
```
#### Flags
```
      --dir string          select migration directory using URL format (default "file://migrations")
      --dir-format string   select migration file format (default "atlas")
      --edit                edit the created migration file(s)

```


### atlas migrate push

Push a migration directory with an optional tag to the Atlas Registry

#### Usage
```
atlas migrate push [flags] directory[:tag]
```

#### Details
The 'atlas migrate push' command allows you to push your migration directory to the Atlas Registry.
If no repository exists in the registry for the pushed directory, a new one is created. Otherwise, the
directory state will be updated. Read more: https://atlasgo.io/registry

#### Example

```
  atlas migrate push --dev-url docker://mysql/8/dev app
  atlas migrate push --env dev app:latest
```
#### Flags
```
      --dev-url string          [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --dir string              select migration directory using URL format (default "file://migrations")
      --dir-format string       select migration file format (default "atlas")
      --lock-timeout duration   set how long to wait for the database lock (default 10s)
      --tag string              tag to push the directory with

```


### atlas migrate rebase

Rebase one or more migration file on top of all other files and update the atlas.sum file.

#### Usage
```
atlas migrate rebase [flags] {name | version}...
```

#### Example

```
  atlas migrate rebase 20060102150405
  atlas migrate rebase 20060102150405 --env dev
  atlas migrate rebase 20060102150405_name.sql --dir file://migrations
```
#### Flags
```
      --dir string          select migration directory using URL format (default "file://migrations")
      --dir-format string   select migration file format (default "atlas")

```


### atlas migrate rm

Remove a migration file from the migration directory. Does not work for remote directories.

#### Usage
```
atlas migrate rm [flags] [amount]
```

#### Example

```
  atlas migrate rm
  atlas migrate rm --env local 20060102150405
  atlas migrate rm --env local 20060102150405_name.sql
```
#### Flags
```
      --dir string          select migration directory using URL format (default "file://migrations")
      --dir-format string   select migration file format (default "atlas")

```


### atlas migrate set

Set the current version of the migration history table.

#### Usage
```
atlas migrate set [flags] [version]
```

#### Details
'atlas migrate set' edits the revision table to consider all migrations up to and including the given version
to be applied. This command is usually used after manually making changes to the managed database.

#### Example

```
  atlas migrate set 3 --url "mysql://user:pass@localhost:3306/"
  atlas migrate set --env local
  atlas migrate set 1.2.4 --url "mysql://user:pass@localhost:3306/my_db" --revision-schema my_revisions
```
#### Flags
```
  -u, --url string                [driver://username:password@address/dbname?param=value] select a resource using the URL format
      --dir string                select migration directory using URL format (default "file://migrations")
      --dir-format string         select migration file format (default "atlas")
      --revisions-schema string   name of the schema the revisions table resides in

```


### atlas migrate status

Get information about the current migration status.

#### Usage
```
atlas migrate status [flags]
```

#### Details
'atlas migrate status' reports information about the current status of a connected database compared to the migration directory.

#### Example

```
  atlas migrate status --url "mysql://user:pass@localhost:3306/"
  atlas migrate status --url "mysql://user:pass@localhost:3306/" --dir "file:///path/to/migration/directory"
```
#### Flags
```
  -u, --url string                [driver://username:password@address/dbname?param=value] select a resource using the URL format
      --format string             Go template to use to format the output
      --dir string                select migration directory using URL format (default "file://migrations")
      --dir-format string         select migration file format (default "atlas")
      --revisions-schema string   name of the schema the revisions table resides in

```


### atlas migrate test

Run migration tests against the given directory

#### Usage
```
atlas migrate test [flags] [paths]
```

#### Example

```
  atlas migrate test --dev-url docker://mysql/8/dev --dir file://migrations .
  atlas migrate test --env dev ./tests
```
#### Flags
```
      --dev-url string            [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --dir string                select migration directory using URL format (default "file://migrations")
      --dir-format string         select migration file format (default "atlas")
      --revisions-schema string   name of the schema the revisions table resides in
      --run string                run only tests matching regexp

```


### atlas migrate validate

Validates the migration directories checksum and SQL statements.

#### Usage
```
atlas migrate validate [flags]
```

#### Details
'atlas migrate validate' computes the integrity hash sum of the migration directory and compares it to the
atlas.sum file. If there is a mismatch it will be reported. If the --dev-url flag is given, the migration
files are executed on the connected database in order to validate SQL semantics.

#### Example

```
  atlas migrate validate
  atlas migrate validate --dir "file:///path/to/migration/directory"
  atlas migrate validate --dir "file:///path/to/migration/directory" --dev-url "docker://mysql/8/dev"
  atlas migrate validate --env dev --dev-url "docker://postgres/15/dev?search_path=public"
```
#### Flags
```
      --dev-url string      [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --dir string          select migration directory using URL format (default "file://migrations")
      --dir-format string   select migration file format (default "atlas")

```


## atlas schema

Work with atlas schemas.

#### Usage
```
atlas schema
```

#### Details
The `atlas schema` command groups subcommands working with declarative Atlas schemas.

#### Flags
```
  -c, --config string        select config (project) file using URL format (default "file://atlas.hcl")
      --env string           set which env from the config file to use
      --var <name>=<value>   input variables (default [])

```


### atlas schema apply

Apply an atlas schema to a target database.

#### Usage
```
atlas schema apply [flags]
```

#### Details
'atlas schema apply' plans and executes a database migration to bring a given
database to the state described in the provided Atlas schema. Before running the
migration, Atlas will print the migration plan and prompt the user for approval.

The schema is provided by one or more URLs (to a HCL file or 
directory, database or migration directory) using the "--to, -t" flag:
  atlas schema apply -u URL --to "file://file1.hcl" --to "file://file2.hcl"
  atlas schema apply -u URL --to "file://schema/" --to "file://override.hcl"

As a convenience, schema URLs may also be provided via an environment definition in
the project file (see: https://atlasgo.io/cli/projects).

If run with the "--dry-run" flag, atlas will exit after printing out the planned
migration.

#### Example

```
  atlas schema apply -u "mysql://user:pass@localhost/dbname" --to "file://atlas.hcl"
  atlas schema apply -u "mysql://localhost" --to "file://schema.sql" --dev-url "docker://mysql/8/dev"
  atlas schema apply --env local --dev-url "docker://postgres/15/dev?search_path=public" --dry-run
  atlas schema apply -u "sqlite://file.db" --to "file://schema.sql" --dev-url "sqlite://dev?mode=memory"
```
#### Flags
```
  -u, --url string        [driver://username:password@address/dbname?param=value] select a resource using the URL format
      --to strings        [driver://username:password@address/dbname?param=value] select a desired state using the URL format
      --exclude strings   list of glob patterns used to filter resources from applying
  -s, --schema strings    set schema names
      --dev-url string    [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --dry-run           print SQL without executing it
      --auto-approve      apply changes without prompting for approval
      --format string     Go template to use to format the output
      --tx-mode string    set transaction mode [none, file] (default "file")
      --plan string       URL to a pre-planned migration (e.g., atlas://repo/plans/name)
      --edit              open the generated SQL in an editor

```


### atlas schema clean

Removes all objects from the connected database.

#### Usage
```
atlas schema clean [flags]
```

#### Details
'atlas schema clean' drops all objects in the connected database and leaves it in an empty state.
As a safety feature, 'atlas schema clean' will ask for confirmation before attempting to execute any SQL.

#### Example

```
  atlas schema clean -u "mysql://user:pass@localhost:3306/dbname"
  atlas schema clean -u "mysql://user:pass@localhost:3306/"
```
#### Flags
```
  -u, --url string      [driver://username:password@address/dbname?param=value] select a resource using the URL format
      --dry-run         print SQL without executing it
      --format string   Go template to use to format the output
      --auto-approve    apply changes without prompting for approval

```


### atlas schema diff

Calculate and print the diff between two schemas.

#### Usage
```
atlas schema diff [flags]
```

#### Details
'atlas schema diff' reads the state of two given schema definitions, 
calculates the difference in their schemas, and prints a plan of
SQL statements to migrate the "from" database to the schema of the "to" database.
The database states can be read from a connected database, an HCL project or a migration directory.

#### Example

```
  atlas schema diff --from "mysql://user:pass@localhost:3306/test" --to "file://schema.hcl"
  atlas schema diff --from "mysql://user:pass@localhost:3306" --to "file://schema_1.hcl" --to "file://schema_2.hcl"
  atlas schema diff --from "mysql://user:pass@localhost:3306" --to "file://migrations" --format '{{ sql . "  " }}'
```
#### Flags
```
  -f, --from strings      [driver://username:password@address/dbname?param=value] select a resource using the URL format
      --to strings        [driver://username:password@address/dbname?param=value] select a desired state using the URL format
      --dev-url string    [driver://username:password@address/dbname?param=value] select a dev database using the URL format
  -s, --schema strings    set schema names
      --exclude strings   list of glob patterns used to filter resources from applying
      --format string     Go template to use to format the output
  -w, --web               open the schema diff ERD in the browser

```


### atlas schema fmt

Formats Atlas HCL files

#### Usage
```
atlas schema fmt [path ...]
```

#### Details
'atlas schema fmt' formats all ".hcl" files under the given paths using
canonical HCL layout style as defined by the github.com/hashicorp/hcl/v2/hclwrite package.
Unless stated otherwise, the fmt command will use the current directory.

After running, the command will print the names of the files it has formatted. If all
files in the directory are formatted, no input will be printed out.



### atlas schema inspect

Inspect a database and print its schema in Atlas DDL syntax.

#### Usage
```
atlas schema inspect [flags]
```

#### Details
'atlas schema inspect' connects to the given database and inspects its schema.
It then prints to the screen the schema of that database in Atlas DDL syntax. This output can be
saved to a file, commonly by redirecting the output to a file named with a ".hcl" suffix:

  atlas schema inspect -u "mysql://user:pass@localhost:3306/dbname" > schema.hcl

This file can then be edited and used with the `atlas schema apply` command to plan
and execute schema migrations against the given database. In cases where users wish to inspect
all multiple schemas in a given database (for instance a MySQL server may contain multiple named
databases), omit the relevant part from the url, e.g. "mysql://user:pass@localhost:3306/".
To select specific schemas from the databases, users may use the "--schema" (or "-s" shorthand)
flag.
	

#### Example

```
  atlas schema inspect -u "mysql://user:pass@localhost:3306/dbname"
  atlas schema inspect -u "mariadb://user:pass@localhost:3306/" --schema=schemaA,schemaB -s schemaC
  atlas schema inspect --url "postgres://user:pass@host:port/dbname?sslmode=disable"
  atlas schema inspect -u "sqlite://file:ex1.db?_fk=1"
```
#### Flags
```
  -u, --url string        [driver://username:password@address/dbname?param=value] select a resource using the URL format
      --dev-url string    [driver://username:password@address/dbname?param=value] select a dev database using the URL format
  -s, --schema strings    set schema names
      --exclude strings   list of glob patterns used to filter resources from applying
      --format string     Go template to use to format the output
  -w, --web               open the schema ERD in the browser

```


### atlas schema plan

Plan a declarative migration for the schema transition

#### Usage
```
atlas schema plan [flags] [name]
```

#### Details
The 'atlas schema plan' command allows users to pre-plan, review, and approve migrations before
executing 'atlas schema apply' on the database. This enables users to preview and modify SQL changes,
involve team members in the review process, and ensure that no human intervention is required during
the atlas schema apply phase. Read more: https://atlasgo.io/declarative/plan

#### Example

```
  atlas schema plan --env dev
  atlas schema plan --env dev --to file://schema.hcl
  atlas schema plan --env dev --from file://schema.hcl
  atlas schema plan --env dev --pending
```
#### Flags
```
      --from strings            URL(s) of the current schema state
      --to strings              URL(s) of the desired schema state
      --auto-approve            approve the plan without asking for confirmation
      --repo string             URL to the schema repository
      --format string           output format
      --dev-url string          [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --lock-timeout duration   set how long to wait for the database lock (default 10s)
      --dry-run                 print the plan and exit
      --push                    push the plan to the registry
      --save                    save the plan to a file
      --edit                    edit the plan in the terminal editor
      --name string             plan name as stored in the registry
      --pending                 push the plan in a pending state
  -o, --output string           output file path (e.g., path/to/file.plan.hcl)

```


#### atlas schema plan approve

Approve a migration plan by its registry URL

#### Usage
```
atlas schema plan approve [flags]
```

#### Example

```
  atlas schema plan approve --url atlas://<schema-slug>/plans/<name>
```
#### Flags
```
      --url string      URL to the plan file in the registry
      --format string   output format

```


#### atlas schema plan lint

Run analysis (migration linting) on a plan file

#### Usage
```
atlas schema plan lint [flags]
```

#### Flags
```
      --from strings            URL(s) of the current schema state
      --to strings              URL(s) of the desired schema state
      --auto-approve            approve the plan without asking for confirmation
      --repo string             URL to the schema repository
      --format string           output format
      --dev-url string          [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --lock-timeout duration   set how long to wait for the database lock (default 10s)
  -f, --file string             URL to the plan file (e.g., file://path/to/file.plan.hcl)

```


#### atlas schema plan list

List all plans in the registry for the given schema transition

#### Usage
```
atlas schema plan list [flags]
```

#### Example

```
  atlas schema plan list --to file://schema.hcl --env dev
```
#### Flags
```
      --from strings            URL(s) of the current schema state
      --to strings              URL(s) of the desired schema state
      --auto-approve            approve the plan without asking for confirmation
      --repo string             URL to the schema repository
      --format string           output format
      --dev-url string          [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --lock-timeout duration   set how long to wait for the database lock (default 10s)
      --pending                 list only pending plans

```


#### atlas schema plan new

Create a new plan file for the schema transition

#### Usage
```
atlas schema plan new [flags]
```

#### Flags
```
      --from strings            URL(s) of the current schema state
      --to strings              URL(s) of the desired schema state
      --auto-approve            approve the plan without asking for confirmation
      --repo string             URL to the schema repository
      --format string           output format
      --dev-url string          [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --lock-timeout duration   set how long to wait for the database lock (default 10s)
      --edit                    edit the plan in the terminal editor
      --name string             plan name as stored in the registry
  -o, --output string           output file path (e.g., path/to/file.plan.hcl)

```


#### atlas schema plan pull

Pull a plan file from the registry

#### Usage
```
atlas schema plan pull [flags]
```

#### Flags
```
      --url string   URL to the plan file in the registry

```


#### atlas schema plan push

Push a plan file to the registry

#### Usage
```
atlas schema plan push [flags]
```

#### Example

```
  atlas schema plan push --file file://plan.hcl
  atlas schema plan push --from file://schema.hcl --to file://schema.hcl --file file://plan.hcl
```
#### Flags
```
      --from strings            URL(s) of the current schema state
      --to strings              URL(s) of the desired schema state
      --auto-approve            approve the plan without asking for confirmation
      --repo string             URL to the schema repository
      --format string           output format
      --dev-url string          [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --lock-timeout duration   set how long to wait for the database lock (default 10s)
  -f, --file string             URL to the plan file (e.g., file://path/to/file.plan.hcl)
      --pending                 push the plan in a pending state

```


#### atlas schema plan validate

Validate a plan file against the schema transition

#### Usage
```
atlas schema plan validate [flags]
```

#### Example

```
  atlas schema plan validate --file file://plan.hcl
  atlas schema plan validate --from file://schema.hcl --to file://schema.hcl --file file://plan.hcl
```
#### Flags
```
      --from strings            URL(s) of the current schema state
      --to strings              URL(s) of the desired schema state
      --auto-approve            approve the plan without asking for confirmation
      --repo string             URL to the schema repository
      --format string           output format
      --dev-url string          [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --lock-timeout duration   set how long to wait for the database lock (default 10s)
  -f, --file string             URL to the plan file (e.g., file://path/to/file.plan.hcl)

```


### atlas schema push

Push the schema with an optional tag to the Atlas Registry

#### Usage
```
atlas schema push [flags] schema[:tag]
```

#### Details
The 'atlas schema push' command allows you to push your schema definition to the Atlas Registry.
If no repository exists in the registry for the schema, a new one is created. Otherwise, a new
version is generated. Read more: https://atlasgo.io/registry

#### Example

```
  atlas schema push --dev-url docker://mysql/8/dev --url file://schema.hcl app
  atlas schema push --env dev app
  atlas schema push --env dev app --version 20230102150405
```
#### Flags
```
  -u, --url strings             desired schema URL(s) to push
      --dev-url string          [driver://username:password@address/dbname?param=value] select a dev database using the URL format
      --version string          version of the schema to push. Defaults to the current timestamp
      --desc string             description of the pushed schema version
      --tag string              tag to push the directory with
      --format string           Go template to use to format the output
      --lock-timeout duration   set how long to wait for the database lock (default 10s)

```


### atlas schema test

Run schema tests against the desired schema

#### Usage
```
atlas schema test [flags] [paths]
```

#### Example

```
  atlas schema test --dev-url docker://mysql/8/dev --url file://schema.hcl .
  atlas schema test --env dev ./tests
```
#### Flags
```
      --dev-url string   [driver://username:password@address/dbname?param=value] select a dev database using the URL format
  -u, --url strings      desired schema URL(s) to test
      --run string       run only tests matching regexp

```


## atlas version

Prints this Atlas CLI version information.

#### Usage
```
atlas version
```



