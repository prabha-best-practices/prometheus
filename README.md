# Monitoring Data: Timing and Interruptions

Monitoring data flows through several segments on its path from a client to Prometheus. Timing interactions along this path can affect the data. Some segments of this path retain data across interruptions, while others do not retain data.

## Best Practice for Intervals

Default values for the monitoring interval, heartbeat interval, and scraping interval are a good choice for many applications. 

If your enterprise needs a different set of values, follow these guidelines for best results:

 + Set the client monitoring sample interval and the client-server heartbeat interval to identical values.
 
 + Set the Prometheus scrape interval to approximately half of the client-server heartbeat interval.

If it is critical to your enterprise that Prometheus capture all monitoring data from these processes, set the Prometheus scrape interval to 1 second. If your enterprise can accept coarser granularity, discarding some of the data samples from these processes, then use a larger scrape interval.

## Client

Most monitoring data begins in a client process. Each client gathers data and transfers it to the realm server in its regular heartbeat messages.

For the most complete results, ensure that the client monitoring sample interval is identical to the client-server heartbeat interval. (You can configure these intervals as global realm properties of the realm server. However, the monitoring sample interval for persistence servers and TIBCO eFTL servers is fixed.)

If the heartbeat interval is longer than the sample interval, the client accumulates data from several sample intervals, and sends the accumulated samples to the realm server in the next heartbeat.

If the client cannot communicate with the realm server, the client retains its monitoring data until it successfully reconnects to the server, then it transfers the accumulated samples. Memory availability could limit the amount of data that a client can accumulate.

When a client stops or restarts, it does not preserve monitoring data it has not yet sent to the realm server.

## Realm Server

The realm server receives data from clients, and also generates monitoring data that measures its own operations. The realm server publishes a stream of monitoring data messages on the built-in monitoring endpoint.

As each monitoring sample set arrives from a client, the realm server immediately publishes it on the monitoring stream. The realm server also stores the new data samples, overwriting previous data samples for that client.

The realm server publishes each sample in a separate message.

The monitoring message stream is available only to current subscribers. Subscribers cannot receive any messages that the realm server sends while a subscribing process is not running, or while the communications link is interrupted. Monitoring data in those messages is no longer available to the subscriber.

You cannot associate a persistence store with the monitoring endpoint.

## Adapter

The adapter (tibpromgateway) subscribes to the monitoring message stream.

As each message arrives, the adapter immediately posts the data to the Prometheus Pushgateway and then discards the message.

The adapter cannot receive any messages that the realm server sends while the adapter is not running, or while the communications link from the realm server is interrupted. Monitoring data in those messages is no longer available to the adapter.

If the adapter process stops, or if communications from the realm server are interrupted, then it cannot receive any messages that the realm server sent in the interim. Monitoring data in those messages is lost.

## Pushgateway

The adapter transfers data to the Prometheus Pushgateway. The Pushgateway behaves as a single-record last value cache. Upon receiving a monitoring sample, it stores the inbound data for scraping. For each client and metric, Pushgateway retains only the most recent data sample.

Monitoring data is lost if the Pushgateway process stops or communications from the adapter are interrupted.

## Prometheus Server

Prometheus scrapes data from the Pushgateway at regular intervals, and stores it for analysis and notification.

If the scrape interval is too long, the Pushgateway could receive data from several sampling intervals before the next scrape. Each sample posted to Pushgateway overwrites the previous sample for the same client and metric, so Prometheus scrapes only the newest sample for each metric, missing older samples.

For the most complete data sets, ensure that the Prometheus scrape interval is approximately half of the realm server's client-server heartbeat interval. In this way, Prometheus scrapes about twice as fast as the monitoring messages arrive, so it scrapes each sample at least once before the next sample overwrites it. Set the parameter scrape_interval in the Prometheus configuration file.

