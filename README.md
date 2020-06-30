# paas-telegraf-buildpack

Cloudfoundry buildpack for deploying a [telegraf][] sidecar
alongside your applications to ingest logs and metrics into
an InfluxDB service instance.

Designed for use with GOV.UK PaaS InfluxDB services.

Use in combination with [paas-chronograf-buildpack][] and
[paas-kapacitor-buildpack][] for visualizing and alerting on
collected metrics.

## Usage

Add the paas-telegraf-buildpack into the list of buildpacks in each application manifest you want to collect metrics for.

The buildpack should NOT be last buildpack.

Example:

```yaml
---
applications:

- name: paas-go-example
  memory: 64M
  instances: 1

  buildpacks:

    # required: add the telegraf sidecar
    - https://github.com/chrisfarms/paas-telegraf-buildpack

    # add your application buildpacks after the sidecar
    - go_buildpack

  env:

    # optional: provide cloudfoundry creds to collect logs
    # and metrics from the cloudfoundry platform
    CF_USERNAME: ((cf_username))
    CF_PASSWORD: ((cf_password))

    # optional: provide a url to collect custom metrics exposed
    # by your application in prometheus compatible format
    PROMETHEUS_METRICS_URL: http://localhost:8080/metrics

    # include your own env vars...
    GOPACKAGENAME: github.com/alphagov/paas-go-example
    GOVERSION: go1.13

  services:

    # required: bind to an influxdb service for metric storage
    - my-influx-db-service
```

## Configuration

### Metric output configuration

Binding an `influxdb` service instance to an application with the telegraf
sidecar will configure telegraf to output measurements to the bound influxdb
database.

### Platform metric collection

When a `CF_USERNAME/CF_PASSWORD` or `CF_CLIENT_ID/CF_CLIENT_SECRET` combination
is provided, then application events, metrics and logs will be read from the
cloudfoundry platform and stored.

You will find application metrics stored in the `cloudfoundry` influxdb measurement.

You will find applicaiton logs stored in the `syslog` influxdb measurement.

### Custom metric collection

If your application exposes custom metrics in a Prometheus format you can
enable collection from the application instance by setting
`PROMETHEUS_METRICS_URL=http://localhost:8080/metrics`

### System metric collection

By default `cpu`, `system`, `net`, `mem` telegraf inputs are enable.

### Postgres metric collection

Binding a `postgres` service instance to the application with a telegraf
sidecar will enable the postgres metric input.

## Note:

:warning: currently deploys a version of telegraf compiled from this PR: https://github.com/influxdata/telegraf/pull/7773

[telegraf]: https://docs.influxdata.com/telegraf/v1.14/
[paas-chronograf-buildpack]: https://github.com/chrisfarms/paas-chronograf-buildpack
[paas-kapacitor-buildpack]: https://github.com/chrisfarms/paas-kapacitor-buildpack
