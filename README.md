<img src="icon.png" alt="HerokuDB Icon" width="72"/>

HerokuDB
========

*Ruby script for downloading and restoring Postgres backups made by the Heroku PG Backups addon*

Installation
------------
To install this script to `/usr/local/bin`, run:

```shell
./herokudb script
```

About
-----
Heroku makes daily backups of your databases, you run a local Postgres
database to experiment with, but how to get the backup to your local
machine, conveniently? Use this script!

## Getting the backup restored locally

Enable the 'PG Backups' addon, e.g. using `heroku addons:add pgbackups`.

Typical use takes three commands:

    herokudb download -a my_app
    herokudb restore -a my_app -d my_database
    herokudb deploy -d my_database

1. The first line will download the database backup to the `tmp/` folder,
   after which no internet connectivity is required anymore.
2. The second will restore the database to `my_database_tempate`, turning
   the archive file into a live database.
3. The third will recreate `my_database` using the template, usually a
   quick final step compared to the restore process.

## Tweaking the process

To restore while downloading (and speed up the process), use the 'pipe'
action instead of 'download' and 'restore':

    herokudb pipe -a my_app -d my_database
    herokudb deploy -d my_database

In order to have minimal downtime on the local database, we use a template
database for restoring (and piping). The replacing of the database content
is done with the `deploy` action, which is therefore separate from the
`restore` or `pipe` action. To disable the use of a template database (and
further speed up the process), use the `-n` option.

    herokudb restore -n -a my_app -d my_database
    herokudb pipe -n -a my_app -d my_database

The restoring of a database dumb is usually quite a cpu- and disk-intensive
operation. By default the restore and pipe action use the `nice -n 20`
utility to minimize this impact. Use the `-p` option to run these actions
on normal priority:

    herokudb restore -p -a my_app -d my_database
    herokudb pipe -p my_app -d my_database

So in short, the fastest way to download, restore and deploy a database is
by using the pipe action in combination with `-n` and `-p`:

    herokudb pipe -n -p -a my_app -d my_database

However in practice, it is usually more convenient to take the two-step
approach:

    herokudb pipe -a my_app -d my_database --verbose
    herokudb deploy -d my_database --verbose

## Checking for new backups

To get a list of all remote and local backups, use the `list` action:

    herokudb list -a my_app

We prevent double-downloading of backups by keeping track of heroku's
database IDs. To force the re-download of a backup use the `-f` option:

    herokudb download -f -a my_app
    herokudb pipe -f my_app -d my_database

To check if the latest backup is already downloaded, use the `available`
action:

    herokudb available -a my_app

## Accessing older backups

By default, we download and restore the latest backup. To indicate a
specific older database, use the `-i` option to specify the database ID:

    herokudb download -i a98 -a my_app

## Filtering dump data

To skip certain parts of the database dump during the restoring of it,
use the `-s` option, followed by a regex. For example to skip all indexes
of a table, use:

    herokudb restore -s '/^CREATE INDEX.*ON my_table USING.*;$/d' -a my_app -d my_database

To filter using this regex, we first decompress the dump and then run it
through the `sed` command. The use of `-s` can have a significant impact
on performance.

## Skipping backups

Heroku's backups are an efficient way to restore databases locally. They
however require a backup to be present. To skip backups and get a direct
copy of the remote database, use the pull action:

    herokudb pull -a my_app -d my_database

Note that pulling skips remote and local backups, so it's mainly useful
for impulsive needs. Creating a remote backup first makes for a much more
reliable workflow.

## Installing herokudb script

For convenience, install this script to your `/usr/local/bin` folder. Use
the `script` action:

    herokudb script

Optionally add `-b /somewhere/else` to install it somewhere else.

## Resuming failed download

By default, failed downloads are resumed on the next attempt. This only
applies to the `download` action; `pipe` actions always start from scratch.
A download can be forced to start from scratch by adding `-f`.

## Using the Heroku Accounts plugin

If you're accessing multiple accounts on Heroku using the accounts plugin,
then make sure to add the `-c` option to provide the account name.

    herokudb pipe -a my_app -d my_database -c my_account

## License

HerokuDB is licensed under the terms of the MIT License.

Made by @leonardvandriel. Copyright (c) 2013 Wit Dot Media Berlin GmbH.
