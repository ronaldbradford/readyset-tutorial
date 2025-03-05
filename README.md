# Readyset Tutorials

This repository provides several reproducible cut/paste tutorials to demonstrate the functionality for the next-generation caching product [Readyset](https://readyset.io) for [MySQL](https://www.mysql.com) and [PostgreSQL](https://postgres.org) databases.

# Tutorial Pre-requisites

- A MySQL database server running Version 8.0+ See list of [Supported Database Versions](https://readyset.io/docs/get-started/install-rs).
- (Optional) A PostgreSQL database server running Versions 13.0+.
- An example dataset. You can find here a number of [preconfigured MySQL Sample Data Sets](https://github.com/ronaldbradford/data/tree/main/mysql-data)
- Example SQL queries for your dataset.
- An [installed Readyset server](https://readyset.io/docs/get-started/install-rs/binaries/install-package) witch is configured to use the MySQL database server. 

# Readyset Setup

A number of `SHOW` commands are available to view information about your Readyset installation using the `mysql` or `psql` clients. For more details see these [SHOW](show/README.md) examples.

# Caching Queries in Readyset

After the installation and configuration of Readyset, you can cache supported queries. See [Caching](caching/README.md).

# Readyset Management

See the commands to start, stop, monitor and configure Readyset in the [Server Commands](server/README.md).
