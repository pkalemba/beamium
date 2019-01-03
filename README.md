# Beamium - metrics scraper for Warp10 & Prometheus

[![GitHub release](https://img.shields.io/github/release/ovh/beamium.svg)]()
[![Build Status](https://travis-ci.org/ovh/beamium.svg?branch=master)](https://travis-ci.org/ovh/beamium)

Beamium collect metrics from HTTP endpoints like http://127.0.0.1/metrics and supports Prometheus and Warp10/Sensision format. Once scraped, Beamium can filter and forward data to a Warp10 Time Series platform. While acquiring metrics, Beamium uses DFO (Disk Fail Over) to prevent metrics loss due to eventual network issues or unavailable service.

Beamium is written in Rust to ensure efficiency, a very low footprint and deterministic performances.

Beamium key points:
 - **Simple**: Beamium is a single binary that does one thing : scraping then pushing.
 - **Integration**: Beamium fetch Prometheus metrics and so benefits from a large community.
 - **Reliable**: Beamium handle network failure. Never loose data. We guarantee void proof graph ;)
 - **Versatile**: Beamium can also scrape metrics from a directory.
 - **Powerful**: Beamium is able to filter metrics and send them to multiple Warp10 platforms.

## How it works?

Scraper (optionals) will collect metrics from defined endpoints. They will store them into the source_dir.
Beamium will read files inside source_dir, and will fan out them according to the provided selector into sink_dir.
Finaly Beamium will push files from the sink_dir to the defined sinks.

The pipeline can be describe this way :

    HTTP /metrics endpoint --scrape--> source_dir --route--> sink_dir --forward--> warp10

It also means that given your need, you could produce metrics directly to source/sink directory, example :

    $ TS=`date +%s` && echo $TS"000000// metrics{} T" >> /opt/beamium/data/sources/prefix-$TS.metrics

## Status
Beamium is currently under development.

## Install
We provide deb packages for Beamium!
```
sudo apt-get install apt-transport-https
sudo lsb_release -a | grep Codename | awk '{print "deb https://last-public-ovh-metrics.snap.mirrors.ovh.net/debian/ " $2 " main"}' >> /etc/apt/sources.list.d/beamium.list
curl https://last-public-ovh-metrics.snap.mirrors.ovh.net/pub.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install beamium
```

## Building
Beamium is pretty easy to build.
 - Clone the repository
 - Setup a minimal working config (see below)
 - Install rust compile tools with `curl https://sh.rustup.rs -sSf | sh`
 - Then `source ~/.cargo/env`
 - Build with `cargo build`
 - Finally, run `cargo run`

If you have already rust:
 - `cargo install --git https://github.com/ovh/beamium`

## Configuration
Beamium come with a [sample config file](config.sample.yaml). Simply copy the sample to *config.yaml*, replace `WARP10_ENDPOINT` and `WARP10_TOKEN`, launch Beamiun and you are ready to go!

### Definitions
Config is composed of four parts:

#### Scrapers
Beamium can have none to many Prometheus or Warp10/Sensision endpoints. A *scraper* is defined as follow:
``` yaml
scrapers:                              # Scrapers definitions (Optional)
  scraper1:                            # Source name                  (Required)
    url: http://127.0.0.1:9100/metrics # Prometheus endpoint          (Required)
    period: 10000                      # Polling interval(ms)         (Required)
    format: prometheus                 # Polling format               (Optional, default: prometheus, value: [prometheus, sensision])
    labels:                            # Labels definitions           (Optional)
      label_name: label_value          # Label definition             (Required)
    filtered_labels:                   # filtered labels              (optional)
      - jobid                          # key label which is removed   (required)
    metrics:                           # filter fetched metrics       (optional)
      - node.*                         # regex used to select metrics (required)
    headers:                           # Add custom header on request (Optional)
      X-Toto: tata                     # list of headers to add       (Optional)
      Authorization: Basic XXXXXXXX
```

#### Sinks
Beamium can have none to many Warp10 endpoints. A *sink* is defined as follow:
``` yaml
sinks: # Sinks definitions (Optional)
  source1:                             # Sink name                                (Required)
    url: https://warp.io/api/v0/update # Warp10 endpoint                          (Required)
    token: mywarp10token               # Warp10 write token                       (Required)
    token-header: X-Custom-Token       # Warp10 token header name                 (Optional, default: X-Warp10-Token)
    selector: metrics.*                # Regex used to filter metrics             (Optional, default: None)
    ttl: 3600                          # Discard file older than ttl (seconds)    (Optional, default: 3600)
    size: 1073741824                   # Discard old file if sink size is greater (Optional, default: 1073741824)
    parallel: 1                        # Send parallelism                         (Optional, default: 1)
    keep-alive: 1                      # Use keep alive                           (Optional, default: 1)
```

#### Labels
Beamium can add static labels to collected metrics. A *label* is defined as follow:
``` yaml
labels: # Labels definitions (Optional)
  label_name: label_value # Label definition             (Required)
```

#### Parameters
Beamium can be customized through parameters. See available parameters bellow:
``` yaml
parameters: # Parameters definitions (Optional)
  source-dir: sources   # Beamer data source directory                     (Optional, default: sources)
  sink-dir: sinks       # Beamer data sink directory                       (Optional, default: sinks)
  scan-period: 1000     # Delay(ms) between source/sink scan               (Optional, default: 1000)
  batch-count: 250      # Maximum number of files to process in a batch    (Optional, default: 250)
  batch-size: 200000    # Maximum batch size                               (Optional, default: 200000)
  log-file: beamium.log # Log file                                         (Optional, default: beamium.log)
  log-level: 4          # Log level                                        (Optional, default: info)
  timeout: 500          # Http timeout (seconds)                           (Optional, default: 500)
  router-parallel: 1    # Routing threads                                  (Optional, default: 1)
  backoff:              # Backoff configuration - slow down push on errors (Optional)
    initial: 500ms        # Initial interval                               (Optional, default: 500ms)
    max: 1m               # Max interval                                   (Optional, default: 1m)
    multiplier: 1.5       # Interval multiplier                            (Optional, default: 1.5)
    randomization: 0.3    # Randomization factor - delay = interval * 0.3  (Optional, default: 0.3)
```

## Contributing
Instructions on how to contribute to Beamium are available on the [Contributing][Contributing] page.

## Get in touch

- Twitter: [@notd33d33](https://twitter.com/notd33d33)

[Contributing]: CONTRIBUTING.md
