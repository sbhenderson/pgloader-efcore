# PGLoader

This is a dedicated fork of [pgloader](https://github.com/dimitri/pgloader) specifically for situations where you are using EF Core migrated databases and want to transfer data only while resetting sequences using pgloader without creating schema.

I am not planning on trying to upstream this as it's not clear to me if there is a generic way of attacking this problem without having a super spider web query (of which I am not qualified to write).

## Example db.load

```text
/* DataService */
load database
    from '/databases/LocalDatabasesGoHere.db'
    into postgresql:///DataBaseGoesHere?sslmode=require
    with truncate, quote identifiers, reset sequences, create no indexes, create no tables, include no drop
    excluding table names like '%EFMigrationsHistory%','EFMigrationsHistory'
    set work_mem to '128MB', maintenance_work_mem to '512 MB';
```

## Theory

With the normal pgloader, reset sequences will not work since it did not create schema. Actually, to be honest, I don't know if that's the absolutely true reason, but when looking at `reset-sequences` and looking at the exact queries used, it is simply not compatible with the structure of the main SQL query because, when transferring without creating schema, the only thing in the `temp table reloids(oid)` are the table names which is not at all correct for `c.oid in (select oid from reloids)`.
