# Readyset Tutorials

This repository provides several reproducible cut/paste tutorials to demonstrate the functionality for [Readyset](https://readyset.io). Readyset the next-generation caching product providing caching support for [MySQL](https://www.mysql.com) and [PostgreSQL](https://postgres.org) databases with no application code changes and no cache invalidation requirements.

# 0. Tutorial Pre-requisites

- A MySQL database server running Version 8.0+.
  - See this list of [Supported Database Versions](https://readyset.io/docs/get-started/install-rs) with Readyset.
- An example dataset.
  - You can find examples of [preconfigured MySQL Sample Data Sets](https://github.com/ronaldbradford/data/tree/main/mysql-data) here.
- Example SQL queries for use with your dataset.

# 1. Readyset Installation and Configuration

There are multiple ways to install Readyset.

1. The [Readyset official documentation](https://readyset.io/docs/get-started/install-rs/binaries/install-package) provides instructions for installing Readyset on Debian-based, Fedora or AL2023 Linux systems which you can use to test with a compatible MySQL or PostgreSQL installation.
2. You can deploy a [docker setup](https://readyset.io/docs/get-started/install-rs/docker) for evaluation purposes.
3. [Readyset Cloud](https://readyset.cloud/) provides you with an AWS Cloud environment were you can test Readyset without an additional resources.
4. You can compile Readyset for example on your Mac OS.

<div style="border: 2px solid green; padding: 10px; background-color: #ccffcc; color: green;">
  A successful installation will have a running Readyset server connected to a target MySQL or PostgresSQL database.
</div>


# 2. Readyset Setup

A number of `SHOW` commands are available to view information about your Readyset installation using the `mysql` or `psql` clients. For more details see these [SHOW](show/README.md) examples.

<div style="border: 2px solid green; padding: 10px; background-color: #ccffcc; color: green;">
  These SHOW commands provide information about the version, status, queries, caches and tables in Readyset.
</div>

# 3. Caching Queries in Readyset

After the installation and configuration of Readyset, with a dataset and example SQL statements you can start caching supported queries. See [Caching](caching/README.md) for several examples.

See the commands to start, stop, monitor and configure Readyset in the [Server Commands](server/README.md).

<div style="border: 2px solid green; padding: 10px; background-color: #ccffcc; color: green;">
  Cached queries provide a much higher lower latency and higher throughput of your SQL statements, and also reduces load on your database server.
</div>

# 4. Evaluating Readyset results

Queries in Readyset are extremely fast. Validating Readyset with the traditional `mysql` client is often not possible. See [Monitoring](monitor/README.md) for different strategies to monitor and track the performance of your application SQL with Readyset.


# 5. Readyset Management

For the commands to start, stop, monitor and configure Readyset, see the [Server Commands](server/README.md) documentation.
