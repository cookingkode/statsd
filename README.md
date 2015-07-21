# StatsD client (Golang)



## Introduction

Go Client library for [StatsD](https://github.com/etsy/statsd/). Contains a direct and a buffered client.
The buffered version will hold and aggregate values for the same key in memory before flushing them at the defined frequency.

This is forked from 
[![GoDoc](https://godoc.org/github.com/quipo/statsd?status.png)](http://godoc.org/github.com/quipo/statsd)

Extensions include :
- Logging of stats in a local file as well as sending to Statsd
- Profiling helper function
- Cleanup of some bugs


## Installation

    go get github.com/cookingkode/statsd

## Supported event types

* Increment - Count occurrences per second/minute of a specific event
* Decrement - Count occurrences per second/minute of a specific event
* Timing - To track a duration event
* Gauge - Gauges are a constant data type. They are not subject to averaging, and they don’t change unless you change them. That is, once you set a gauge value, it will be a flat line on the graph until you change it again
* Absolute - Absolute-valued metric (not averaged/aggregated)
* Total - Continously increasing value, e.g. read operations since boot



## Sample usage

```go
package main

import (
    "time"

	"github.com/quipo/statsd"
)

var stats *statsd.StatsdBuffer

func main() {
	// init
	prefix := "myproject."
	statsdclient := statsd.NewStatsdClient("localhost:8125", prefix)
	statsdclient.CreateSocket()
	interval := time.Second * 2 // aggregate stats and flush every 2 seconds
	stats = statsd.NewStatsdBuffer(interval, statsdclient, "/var/log/local.stats")
	defer stats.Close()

	// not buffered: send immediately
	statsdclient.Incr("mymetric", 4)

	// buffered: aggregate in memory before flushing
	stats.Incr("mymetric", 1)
	stats.Incr("mymetric", 3)
	stats.Incr("mymetric", 1)
	stats.Incr("mymetric", 1)
}

func aFunction(){
	defer stats.TimeThisFunction(time.Now())
	...
}
```

The string "%HOST%" in the metric name will automatically be replaced with the hostname of the server the event is sent from.

