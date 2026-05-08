# Real-time transports (WebSockets, polling, SSE)

**TL;DR:** Three ways for a server to push updates to a client. WebSockets are bidirectional and persistent; SSE is server-to-client streaming over HTTP; long-polling is a workaround if neither is available. Pick the one that matches the data shape and the existing infra.

## What they are

**Long polling** — client makes an HTTP request; server holds it open until there's something to send (or a timeout). On response, client immediately makes another request. Looks like a stream from the client's perspective.

**Server-Sent Events (SSE)** — client opens a special HTTP connection (`Content-Type: text/event-stream`) that the server writes events to. Server-to-client only; native browser support; auto-reconnect built in.

**WebSockets** — full-duplex, persistent TCP connection over a single upgraded HTTP request. Both sides can send messages anytime. Works through proxies (with caveats); supported in browsers and most languages.

## When to reach for each

| Use case | Best fit | Why |
|---|---|---|
| Chat, multiplayer game, collab editor | WebSockets | Bidirectional, low latency, can multiplex many message types |
| Stock-tick / news feed / notifications | SSE | One-way streaming; HTTP-friendly (no proxy issues, no special TCP); auto-reconnect |
| Tail of an ML training job, terminal | SSE | Same as above |
| Rare updates, simplest possible | Long polling | Works through any proxy/firewall; no persistent connections to manage |
| Static or rarely-changing data | Don't push | Just poll on a normal interval, or query on demand |

## What they're NOT good at

- **WebSockets through every proxy.** Many corporate / older HTTP proxies break WebSockets. SSE travels over plain HTTP and avoids this.
- **SSE for client-to-server.** SSE is one-way (server-to-client). If clients also need to send messages, they make separate POST requests — works, but feels awkward vs. WebSocket's symmetry.
- **Long polling at high concurrency.** Each client holds a long-lived HTTP connection — and you have to allocate a worker / thread / event-loop slot to each. WebSockets is more efficient per-connection at scale.
- **Any of these for "tens of millions of concurrent connections."** That's a real engineering problem regardless of transport — connection-per-process limits, OS file descriptor limits, kernel TCP state. Use specialized brokers (Phoenix Channels, Centrifugo, AWS API Gateway WebSockets) at that scale.

## Connection cost at scale

A persistent connection (WebSocket or SSE) costs memory + a file descriptor + (often) a thread or coroutine on the server. At 1M concurrent connections, that's not a single-server problem — it's a fleet plus a connection-aware load balancer (sticky routing) plus a fan-out pattern (pub/sub).

## Alternatives + tradeoffs

| Option | Direction | Connections | Browser support | Pick when |
|---|---|---|---|---|
| Long polling | C↔S (request/response) | Per-poll | Universal | Last-resort compatibility |
| SSE | S→C streaming | Persistent | Native (EventSource API) | Server pushing to clients, no client-to-server stream |
| WebSockets | Bidirectional | Persistent | Native (WebSocket API) | True bidirectional, low latency |
| HTTP/2 server push | S→C | Per-stream | Mostly browser-deprecated | Probably skip — was hyped, mostly didn't work out |
| WebRTC data channels | P2P | Direct | Native | Peer-to-peer, latency-critical (game state, voice/video) |

## Related
- [load-balancer](load-balancer.md)
- [rate-limiting](rate-limiting.md)
