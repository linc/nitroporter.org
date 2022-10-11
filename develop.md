# Developer Guide

Nitro Porter works in this order:

1. The **Source** package translates the data to the intermediary "porter format" (roughly analagous to Vanilla's database schema). These are new database tables with the prefix `PORT_`.
2. The **Target** package translates the data to the final platform format. These can be existing tables from an installation, or it will create them new using the information provided.
3. If a **Postscript** file with the same name as the Target exists, it runs last. This is for doing colcultions that require the data to have been fully transferred already, for example generating data that wasn't ever in the Source.

The `ExportModel` is a utility class that gets passed between every step in the process. It's abbreviated as `$ex` throughout the code.

Use its `comment()` method for logging. Open `porter.log` in your favorite log reader for realtime feedback.

## Add a new Source

**New sources will be automatically detected at runtime and added as options.**

1. Copy and rename `src/Source/Example.php`.
2. Edit the `SUPPORTED` data array, following the inline comments.
3. The basic types of data are stubbed out, one per method. Follow the inline docs.

Source packages are invoked by their `run()` method. It must call any methods you add in the order you want.

### Maps and filters

Use a `$map` array to directly translate a column name in one database to another. Need the data transformed? You can use a function. Pass an array like `['Column' => 'Name', 'Filter' => 'HTMLDecoder']` and the `Name` column's value will be passed to the function `HTMLDecoder()` (along with the rest of the data in the row) for manipulation and the return value will be stored instead of the original. Use the `src/Functions/filter.php` for adding new filters.

### Using export()

The `ExportModel::export()` call is what does the data transfer. The `$query` parameter must at least select all the columns in the `$map` array for it to work. Use the table prefix `:_` for it to be dynamically replaced per the user's input.

### Requiring tables and columns

You can use the `$sourceTables` property to require certain tables and columns in the source database, but it's optional.

## Add a new Target

1. Copy and rename `src/Target/Example.php`.
2. Edit the `SUPPORTED` data array, following the inline comments.
3. The basic types of data are stubbed out, one per method. Follow the inline docs.

Target packages first have their `validate()` method called, then their `run()` method.

To confirm an optional data type exists before importing it, use `targetExists()` on the `ExportModel`.

### Using import()

The `import()` method works a bit more cleanly than the `export()` method. It takes a map array, but also accepts a built SQL statement rather than a query string. It requires defining the target database table structure, and separates filters into their own array for clarity. Use `$ex->dbImport()` to build the SQL statement.

If a column isn't present in the structure passed to `import()`, it will be ignored entirely.

### Using a Postscript

Simply add a new PHP class to the `src/Postscript` folder with the same name as the Target and a method named `run()`. It will automatically run after the import. Its `storage` property will get set with the database connection automatically.


## Working with database connections

Nitro Porter use's the [Laravel Illuminate](https://github.com/illuminate/database) database driver. Refer to its [documentation](https://laravel.com/docs/9.x) for help.

While Nitro Porter reuses an existing database connection wherever possible, it defaults to using an unbuffered query for speed, and it will often be advisable to use the driver's `cursor()` method to stream the results.

You need a second, separate database connection to do other queries while unbuffered results are streaming. The streaming connection is effectively mid-query. While this rarely comes up in **Source** packages since they are simply dumping information and the **Target** package usually abstracts away much of this work, it can get tricky when complex **Postscript** operations are necessary. Refer to the `Flarum` Postscript for examples.