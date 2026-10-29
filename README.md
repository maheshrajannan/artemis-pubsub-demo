# Dapr pub/sub with ActiveMQ Artemis

A small demo of [Dapr pub/sub](https://docs.dapr.io/developing-applications/building-blocks/pubsub/) running on [Apache ActiveMQ Artemis](https://activemq.apache.org/components/artemis/). It's based on the Dapr [pub-sub tutorial](https://github.com/dapr/quickstarts/tree/master/tutorials/pub-sub). The apps are unchanged. Only the broker is different.

Dapr talks to Artemis through its AMQP 1.0 component, `pubsub.amqp`. You'll find it in [`components/pubsub.yaml`](./components/pubsub.yaml).

## What's inside

| App | Role | Topics |
|---|---|---|
| `react-form` | Publisher (web form) | A, B or C |
| `node-subscriber` | Subscriber | A, B |
| `python-subscriber` | Subscriber | A, C |
| `csharp-subscriber` | Subscriber | A, B, C |

## Prerequisites

- [Dapr CLI](https://docs.dapr.io/getting-started/install-dapr-cli/), runtime 1.19 or later (`dapr init`)
- [Docker](https://docs.docker.com/get-docker/)
- Node.js, plus [uv](https://docs.astral.sh/uv/) for Python. .NET 6 or later is only needed for the C# app.

## Run it

1. Start Artemis.

```bash
docker run -d -p 5672:5672 -p 8161:8161 --env ARTEMIS_USER=artemis --env ARTEMIS_PASSWORD=artemis --name artemis apache/activemq-artemis:latest
```

2. Start the Node subscriber.

```bash
cd node-subscriber
npm install
dapr run --app-id node-subscriber --resources-path ../components --app-port 3000 node app.js
```

3. In a new terminal, start the Python subscriber.

```bash
cd python-subscriber
uv sync
dapr run --app-id python-subscriber --resources-path ../components --app-port 5001 uv run python app.py
```

4. Optionally, start the C# subscriber in another terminal.

```bash
cd csharp-subscriber
dapr run --app-id csharp-subscriber --resources-path ../components --app-port 5009 dotnet run csharp-subscriber.csproj
```

5. Publish a message from the repo root.

```bash
dapr publish --publish-app-id node-subscriber --pubsub pubsub --topic A --data-file message_a.json
```

Or start the web form and open http://localhost:8080.

```bash
cd react-form
npm run buildclient
npm install
dapr run --app-id react-form --resources-path ../components --app-port 8080 npm run start
```

## What you should see

These snippets are from a test run on Dapr 1.19.0. Each subscriber only gets the topics it asked for.

When an app starts, Dapr loads the Artemis component and subscribes to the topics.

```text
Component loaded: pubsub (pubsub.amqp/v1)
app is subscribed to the following topics: [[A B]] through pubsub=pubsub
Attempting to subscribe to multicast://A
Attempting to subscribe to multicast://B
```

Publishing A, B and C.

```text
$ dapr publish --publish-app-id node-subscriber --pubsub pubsub --topic A --data-file message_a.json
✅  Event published successfully
```

The Node subscriber gets A and B.

```text
A:  Message on A
B:  Message on B
```

The Python subscriber gets A and C.

```text
Received message "Message on A" on topic "A"
Received message "Message on C" on topic "C"
```

The C# subscriber listens to all three topics.

You can also watch the messages in the Artemis console at http://localhost:8161 (user `artemis`, password `artemis`).

## Clean up

Stop each app with Ctrl+C, then remove the broker.

```bash
docker rm -f artemis
```

## Going back to Redis

Drop `--resources-path ../components` from the `dapr run` commands. Dapr then uses the default Redis component that `dapr init` set up.

## License

Apache 2.0, same as the original [dapr/quickstarts](https://github.com/dapr/quickstarts) repo. See [LICENSE](./LICENSE).
