# influxdb-nagios-plugin

InfluxDB Nagios Plugin

Uses the excellent [nagiosplugin](https://pythonhosted.org/nagiosplugin/) project to create
measurement-based Nagios checks using [InfluxDB](https://influxdb.com/).

[![Build Status](https://travis-ci.org/locationlabs/influxdb-nagios-plugin.png)](https://travis-ci.org/locationlabs/influxdb-nagios-plugin)

## Queries

The plugin issues InfluxDB queries in a few differnt forms based on command line input.

The most common of these (and the query that the tool is built-around) is define by the
`single` sub-command; it will issue a query of the form:

    SELECT time, value FROM <measurement> WHERE time > now() - <age> AND host = '<hostname>'

The plugin then validates that there are a sufficient number of results and that the mean of
these measurements are within acceptable bounds.

This model works well with [Telegraf](https://github.com/influxdb/telegraf) in which healthy
hosts will emit specific measurements regularly. Many of the plugin's defaults are optimized
for this use case.

In 0.10.x, Telegraf changed their model from having point measurements (time, value) to
to collections (time, value, value2).  That is, previously there would be mulitple series such as
`cpu_usage_nice` and `cpu_usage_system`, each containing only (time, value) pairs.  In new
model, the series cpu would contain (time, `usage_nice`, `usage_system`).

## Installation

Use pip:

    pip install influxdbnagiosplugin


## Usage

	Usage: check-measurement [OPTIONS1] COMMAND [OPTIONS2|OPTIONS3]...
	
	Command line entry point. Defines common arguments.
	
	Options 1:
	  -v, --verbose               Can be specified multiple times for increasing verbosity
	  --hostname TEXT             InfluxDB hostname
	  --port INTEGER              InfluxDB port
	  --username TEXT             InfluxDB usernanme
	  --password TEXT             InfluxDB password
	  --database TEXT             InfluxDB database name, default 'telegraf'
	  --count-error-range TEXT    Range of measurement counts that are NOT
								  considered an error
	  --count-warning-range TEXT  Range of measurement counts that are NOT
								  considered a warning
	  --mean-error-range TEXT     Range of measurement means that are NOT
								  considered an error
	  --mean-warning-range TEXT   Range of measurement means that are NOT
								  considered a warning
	  --timeout INTEGER           Timeout in seconds for connecting to InfluxDB
	  --help                      Show this message and exit.
	
	Commands:
	  query   Run an explicit query.
	  single  Run a query for a single measurement.
	
	Usage1: check-measurement [OPTIONS1] single [OPTIONS2] MEASUREMENT HOSTNAME
	
	  Run a query for a single measurement.
	
	Options 2:
	  --age TEXT
	  --where TEXT  Extra where conditions to include.
	  --field TEXT
	  --help        Show this message and exit.
	
	Usage2 : check-measurement [OPTIONS1] query [OPTIONS3] QUERY
	
	  Run an explicit query.
	
	Options 3:
	  --help  Show this message and exit.

## Examples

### Run a query for a single measurement

Run a query for a `cpu` measurement `cpu-total`  the field value `usage_iowait` filtered for host `yourhost`.The options used  before the `single` command and after the `single` command can not be switched.

```shell
$./check-measurement --mean-error-range 10 --mean-warning-range 5 single --field usage_iowait --where cpu='cpu-total' --age 2m cpu yourhost
```

Output:

```shell
MEASUREMENTS OK - cpu is [0.2507941815741449, 0.21819402484046085, 0.19283977530009686, 0.3185247275776435] | count=4;2:;1: mean=0.24508817732308652;5;10
```

### Run an explicit query

Run an explicit query `SELECT time, usage_iowait FROM cpu WHERE time > now() - 2m AND host = 'mo4' AND cpu = 'cpu-total'` 

```shell
$./check-measurement --mean-error-range 10 --mean-warning-range 5 query "SELECT time, usage_iowait FROM cpu WHERE time > now() - 2m AND host = 'yourhost' AND cpu = 'cpu-total'"
```

Output:

```shell
MEASUREMENTS OK - cpu is [0.2507941815741449, 0.21819402484046085, 0.19283977530009686, 0.3185247275776435] | count=4;2:;1: mean=0.24508817732308652;5;10
```

