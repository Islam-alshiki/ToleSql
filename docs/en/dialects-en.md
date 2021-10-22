# Dialects

ToleSql supports different dialects. ToleSql does not generate any literals or keywords in SQL queries, all literals are extracted from the active dialect.

ToleSql supports explicit or implicit syntax to perform JOINs between tables, the explicit form uses the keyword `JOIN` and` ON` and the implicit one separates the data sources by commas in the `FROM` clause.

ToleSql comes with a single dialect, for SqlServer and it is the one that is set by default. The SqlServer dialect uses `explicit joins`.

New dialects can be built by implementing the `ToleSql.Dialect.IDialect` interface or inheriting from` ToleSql.Dialect.DialectBase`. Either incorporating it into the project through a pull request or externally.

As an example you can take a look at [SqlServer dialect](../../src/ToleSql/SqlServer/SqlServerDialect.cs)