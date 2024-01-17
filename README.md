# Migrations.jl

***A Julia package for managing database migrations***

GitHub Actions : [![Build Status](https://github.com/JuliaServices/Migrations.jl/workflows/CI/badge.svg)](https://github.com/JuliaServices/Migrations.jl/actions?query=workflow%3ACI+branch%3Amaster)

[![codecov.io](http://codecov.io/github/JuliaServices/Migrations.jl/coverage.svg?branch=master)](http://codecov.io/github/JuliaServices/Migrations.jl?branch=master)

## Usage

Migrations.jl tries to be simple, transparent, and flexible. It relies on minimal required structure to work, while allowing for more complex setups.

It aims to be compatible with all database packages that support the interfaces in [DBInterface.jl](https://github.com/JuliaDatabases/DBInterface.jl).
Currently that includes SQLite, MySQL, LibPQ.jl, and ODBC.jl.

The primary interface is calling `Migrations.runmigrations(conn::DBInterface.Connection, dir::String)`.

    Migrations.runmigrations(conn::DBInterface.Connection, dir::String)

Using an established database connection `conn` (which should have the appropriate schema already
selected), search the directory `dir` for migration files and apply them to the database. Migration
files should be named like `V1__baseline.sql`, `V2__latlong.sql`, etc. where they _must_ start with
a capital `V` followed by a number, followed by two underscores, followed by a description of the
migration. The number must be unique across all migrations. The description can be anything, but
should be descriptive of the migration. The file extension currently must be `.sql`.

Migration files found in `dir` will be checked against a special `__migrations__` table that
the Migrations.jl package manages in the database connection for tracking which migrations have
already been applied. If a migration file is found in `dir` that has not been applied, it will be
applied to the database. If a migration file is found in `dir` that has already been applied, it
will be skipped. If a migration file is found in `dir` that has been applied but has changed since
it was applied, an error will be thrown (migrations should be immutable once applied).

Migration files may contain multiple SQL statements, separated by semicolons. Each statement will
be executed in order. If any statement fails, the entire migration will be rolled back and an error
will be thrown. If a migration file contains a syntax error, the migration will be rolled back and
an error will be thrown.

## Example

```julia
using Migrations, SQLite

db = SQLite.DB("test.db")

Migrations.runmigrations(db, "migrations")
```
