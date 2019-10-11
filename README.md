[WIP]

# pg_cryogen

`pg_cryogen` is an append-only compressed pluggable storage.

## Dependencies

`pg_cryogen` requires `liblz4` and `libzstd` installed.

## Build

Like with any other postgres extensions after clone just run in command line:

```
make install
```

Or if postgres binaries are not in the `PATH`:

```
make install PG_CONFIG=/path/to/pg_config
```

## Using

`pg_cryogen` implements pluggable storage API and hence requires PostgreSQL 12 or later. To use it as a storage it needs to be specified as an access method while creating a table. E.g.:

```sql
create extension pg_cryogen;
create table my_table (...) using pg_cryogen;
```

Due to the specifics of current implementation of pluggable storage API and storage layout of `pg_cryogen` it was only possible to implement bulk inserts via COPY command. Therefore regular inserts are not implemented.

Both index scan and bitmap scans are implemented as well.

### Compression parameters

`pg_cryogen` offers two compression methods: `lz4`, which provides on average better compression/decompression speed, and `zstd`, which provides better compression ratio. Since pluggable storage API currently does not provide ways to set custom parameters for storages, the only way to set them is by using GUCs:

* `pg_cryogen.compression_method`: possible options are `lz4` (default) and `zstd`.
* `pg_cryogen.lz4_acceleration`: `lz4` specific parameter which specifies how fast compression would be; valid values are [1..50], where 1 is better compression ratio and higher value is better speed (default is 1);
* `pg_cryogen.zstd_compression_level`: `zstd` specific parameter for compression level; valid values are [-5..22] (default is 1).

Those GUCs can be set in either in postgresql.conf or for any particular session.
