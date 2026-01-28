
# Knative Eventing and Serving

## Eventing Video

https://www.cncf.io/online-programs/event-driven-architecture-with-knative-events/

## Concepts

Source:
* Generates or imports events from external sources - to produce the event in CloudEvents format
* Lots of existing sources (community + vendor) - e.g. GitHub, Kafka, CronJobs, etc.
* Can also create custom sources
* Observes and reacts to the fact in the environment or application, and delivers the events to a Broker
* Defined through CRDs, instantiated as Custom Objects (CO)
* Two types:
  * Push - e.g. webhook running as a Knative Service
    * but the service must be accessible (exposed) to the producer
  * Pull - e.g. polling an API

Broker:
* Acts as an event router
* There is a Default Broker
* Also other Brokers, such as Kafka Broker
  * In this case Kafka provides the message transport, and Knative adds the eventing semantics with CloudEvents
* Durability and delivery guarantees depend on the Broker implementation

Trigger:
* Provides the way to subscribe events from a Broker to a Sink:
  * Filering based on CloudEvents attributes

Sink:
* Receives events from triggers
* Sink can be any addressable endpoint (e.g. Knative Service))
* Can be a Knative Service, or other endpoints that can receive CloudEvents

Messaging:
* Channels: Alternative to Brokers with more flexible event routing
* Flows: Sequence vs Parallel

## Tutorial

https://knative.dev/docs/getting-started/tutorial/

### Binaries

* `kn` (client)<br>
  https://github.com/knative/client/releases/tag/knative-v1.19.5

* `quickstart` (plugin)<br>
  https://github.com/knative-extensions/kn-plugin-quickstart/releases/tag/knative-v1.19.4

* `func` (plugin)<br>
  https://github.com/knative/func/releases/tag/knative-v1.19.5

### Create Cluster

kn quickstart kind --kubernetes-version 1.32.8

### Knative Serving

#### Create `func` Project

```bash
kn func create -l go hello
```

#### Run Locally

```bash
cd hello
kn func run --registry docker.io/rconway
```

#### Deploy to Cluster

```bash
kn func deploy
```

#### Build (push but no deploy)

```bash
kn func build
```

#### Deploy Knative Service

```bash
kn service list
```

```bash
kn service create hello \
# --image ghcr.io/knative/helloworld-go:latest \
--image rconway/hello \
--port 8080 \
--env TARGET=World
```

#### New Service Revision

```bash
kn service update hello \
--env TARGET=Knative
```

#### List Revisions

```bash
kn revision list
```

#### Split Traffic

```bash
kn service update hello \
--traffic hello-00001=50 \
--traffic @latest=50
```

### Knative Eventing

#### List Brokers

```bash
kn broker list
```

#### Deploy CloudEvents Player

The CloudEvents Player is a web-based UI for sending and receiving CloudEvents. We use it here as both a source and a sink.

```bash
kn service create cloudevents-player \
--image quay.io/ruben/cloudevents-player:latest
```

Open the player UI:

```bash
xdg-open http://cloudevents-player.default.127.0.0.1.sslip.io/
```

#### Connect Player to the Default Broker

The `example-broker` is created by the `kn quickstart` setup.

> This creates a `SinkBinding` that connects the `cloudevents-player` service to the `example-broker` - so that the player knows where to send events. The `SinkBinding` mutates the `cloudevents-player` service to add the necessary environment variables, such as `K_SINK` etc.

```bash
kn source binding create ce-player-binding \
  --subject "Service:serving.knative.dev/v1:cloudevents-player" \
  --sink broker:example-broker
```

#### Create Trigger

This trigger sends all events from the `example-broker` to the `cloudevents-player` service.

```bash
kn trigger create cloudevents-trigger \
  --sink cloudevents-player \
  --broker example-broker
```

Filtered Trigger:

```bash
kn trigger delete cloudevents-trigger
kn trigger create cloudevents-player-filter \
  --sink cloudevents-player \
  --broker example-broker \
  --filter type=some-type
```
