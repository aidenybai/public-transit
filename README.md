# relayo

A WebSocket relay that connects agent handlers (running in Node.js) with browser clients. Handlers register with the relay server and expose an async generator interface. Browser clients connect over WebSocket to invoke handlers, stream responses, and manage sessions with abort/undo/redo support.

This is useful when you need to bridge a long-running backend process (like an AI agent, build tool, or CLI) with a browser frontend over WebSocket, with built-in session management, reconnection, and multiplexing.

## Install

```bash
npm install relayo
```

## How it works

```
┌─────────┐   WebSocket   ┌──────────────┐   WebSocket   ┌─────────┐
│ Browser  │ ◄───────────► │ Relay Server │ ◄───────────► │ Handler │
│ (client) │               │   (hub)      │               │ (agent) │
└─────────┘               └──────────────┘               └─────────┘
```

1. A **relay server** runs on a known port (default `4722`)
2. **Handlers** connect and register themselves by ID (e.g. `"my-agent"`)
3. **Browser clients** connect and can invoke any registered handler
4. Messages stream back from handler → relay → browser in real time

## Usage

### 1. Start a relay with a handler (Node.js)

The simplest way — `connectRelay` auto-starts a relay server if one isn't already running, then registers your handler:

```ts
import { connectRelay } from "relayo";

const connection = await connectRelay({
  handler: {
    agentId: "my-agent",
    run: async function* (prompt, options) {
      yield { type: "status", content: "Thinking..." };
      // do work...
      yield { type: "done", content: "Here's the result." };
    },
    abort: (sessionId) => {
      // cancel in-progress work
    },
  },
});

// later: connection.disconnect()
```

### 2. Connect from the browser

```ts
import { createRelayClient } from "relayo/client";

const client = createRelayClient();
await client.connect();

// Listen for handler availability
client.onHandlersChange((handlers) => {
  console.log("Available handlers:", handlers);
});

// Send a request
client.sendAgentRequest("my-agent", {
  prompt: "Refactor this component",
  content: ["const App = () => <div>hello</div>"],
});

// Stream responses
client.onMessage((message) => {
  if (message.type === "agent-status") {
    console.log("Status:", message.content);
  } else if (message.type === "agent-done") {
    console.log("Done:", message.content);
  } else if (message.type === "agent-error") {
    console.error("Error:", message.content);
  }
});
```

### 3. Use the high-level agent provider (browser)

For a more ergonomic async-iterable interface:

```ts
import { createRelayClient, createRelayAgentProvider } from "relayo/client";

const client = createRelayClient();
await client.connect();

const agent = createRelayAgentProvider({
  relayClient: client,
  agentId: "my-agent",
});

const controller = new AbortController();

for await (const chunk of agent.send(
  { prompt: "Fix the bug", content: ["..."] },
  controller.signal,
)) {
  console.log(chunk);
}
```

### 4. Standalone relay server (advanced)

If you want to run the relay server separately from handlers:

```ts
import { createRelayServer } from "relayo/server";

const server = createRelayServer({ port: 4722, token: "secret" });
await server.start();

// Handlers and browsers connect over WebSocket
// Token is validated on connection
```

## Exports

| Path              | Platform | Description                                    |
| ----------------- | -------- | ---------------------------------------------- |
| `relayo`          | Node.js  | Everything (server + client + protocol)        |
| `relayo/server`   | Node.js  | `createRelayServer`                            |
| `relayo/client`   | Browser  | `createRelayClient`, `createRelayAgentProvider`|
| `relayo/protocol` | Any      | Shared types and constants                     |

## Authentication

Pass a `token` option to the server and clients. The relay validates the token on both HTTP health checks and WebSocket connections:

```ts
// Server
createRelayServer({ token: "secret" });

// Handler
connectRelay({ handler, token: "secret" });

// Browser
createRelayClient({ token: "secret" });
```

## License

MIT
