# User Guide

## Requirements

You need:

* PHP 8.0+
* MariaDB (or whichever databases your platforms require)
* PHP's PDO driver for your data sources (probably MySQL or PostgreSQL).
* 256MB of memory allocated to PHP

Nitro Porter will set PHP's memory limit to 256MB. If it's unable to do so, it may suffer performance issues or generate errors. For small forums, you may be able to safely reconfigure it to 128MB or lower.

A quick way to get all of the above would be installing MAMP or XAMPP on your laptop. The longer way, if you're doing this often or have huge datasets, is to follow my [PHP localhost guide for Mac](https://lincolnwebs.com/php-localhost/).


## Installation

1. [Get Composer](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos).
2. `composer global require "linc/nitro-porter"`.
3. Copy `config-sample.php` as `config.php`. 
4. Add connections for your source and output to `config.php`.
5. See the options with `porter --help`.


## Basic Usage

### Get oriented

Get the "short" names of the packages and connections you want to use.

Run `porter list` and then choose whether to list:
* sources [`s`] — Package names you can migrate from
* targets [`t`] — Package names you can migrate to
* connections [`c`] — What's in your config (did you make one?)

Note the bolded values without spaces or special characters. Those are the `<name>` values you need next.

### Check support

What can you migrate? Find out!

Run `porter show source <name>` and `porter show target <name>` to see what feature data is supported by the source and target. Data **must be in both** for it to migrate.

### Run the migration

Use `porter run --help` for a full set of options (including shortcodes).

A very simple run might look like: 
```
porter run --source=<name> --input=<connection> --target=<name>
```

**Example A**: Export from Vanilla in `example_db` to Flarum in `test_db`:
```
porter run --source=vanilla2 --input=example_db --target=flarum --output=test_db
```

**Example B**: Export from XenForo in `example_db` to Flarum in the same database, using shortcodes:
```
porter run -s xenforo -i example_db -t flarum
```

## Troubleshooting

### Follow the logs

Nitro Porter logs to `porter.log` in its installation root (e.g. `~/.composer/vendor/linc/nitro-porter` on MacOS). Open it with your favorite log viewer to follow along with its progress.

### Database table prefixes

Try using the same database as both source & target. Nitro Porter works well with multiple platforms installed in the same database using unique table prefixes.

Currently, it **can only use the system default table prefix for targets**, but you can customize the source prefix. It uses `PORT_` as the prefix for its intermediary work storage. You can safely delete the `PORT_` tables after the migration.
