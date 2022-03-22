# Infinitepay Monitoring-queries

## Instalation
There are few dependencies, that you need before you can compile your code:
- Postgresql 9.6
- Cargo 1.51.0-nightly
- Rust 2018

``` sh
cp config/Development.toml.example config/Development.toml

# edit it to match your database credentials
atom config/Development.toml

cargo build
cargo test -- --test-threads=2
```

## Fixtures
The data that we test has two forms of arriving, the furst one is the dump.sql which is load for each test parallelly:

To update the fixtures, from files-api, run:
```
RAILS_ENV=test bundle exec rails db:drop && RAILS_ENV=test bundle exec rails db:create && RAILS_ENV=test bundle exec rails db:schema:load && RAILS_ENV=test bundle exec rails db:migrate && RAILS_ENV=test rails db:fixtures:load
pg_dump YOURFILESTESTDATABASE > ./tests/dump.sql
psql YOURNEWDATABASE < ./tests/dump.sql #keep in mind that the tests does this automatically for you

```

To customize the data creation for testing purposes, use the:

```
custom-testing.sql
```
which will also be loaded at each test run


## Deploying to Grafana

1. Grafana/M3 works by checking what data points is available to be worked with, this code base works as a way to expose points in a timeseries, [read this article](https://www.influxdata.com/time-series-database/) if you need further explanation. [(M3 shares the same interface as InfluxDB and Prometheus, read this article if things don't make sense)](https://aiven.io/blog/an-introduction-to-m3)
2. Consider checking if your data points already exist on M3/Grafana and if they are only needing to be exposed, in this case, check  the Metrics browser on grafana, for example:

![image](https://user-images.githubusercontent.com/53792026/159383205-b4e0a1af-2bab-48ed-8656-8138e4288732.png)

3. If no data point exist for what you want to filter on grafana, create a `data` sink:
   1. The from-sql data structure https://github.com/cloudwalk/infinitepay-monitoring-sink/blob/main/src/data/job_states.rs#L8
   2. The query which will fill the from-sql https://github.com/cloudwalk/infinitepay-monitoring-sink/blob/main/src/data/job_states.rs#L31
   3. the to-M3 (influxdb) data structure https://github.com/cloudwalk/infinitepay-monitoring-sink/blob/main/src/data/job_states.rs#L18
   4. the sink that exports data to using the structure to M3 https://github.com/cloudwalk/infinitepay-monitoring-sink/blob/main/src/data/job_states.rs#L49

4. After deploy, check if data is available in grafana  `gcloud app deploy app-production.yaml --no-promote --project infinitepay-production`
