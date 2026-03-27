# relayo

WebSocket relay for bridging agent handlers with browser clients.

## Install

```bash
pnpm add relayo
```

## Usage

### Server (Node.js)

```ts
import { createRelayServer } from "relayo/server";

const server = createRelayServer({ port: 4722 });
await server.start();
```

### Client (Browser)

```ts
import { createRelayClient } from "relayo/client";

const client = createRelayClient();
await client.connect();
```

### Connection Helper

```ts
import { connectRelay } from "relayo";

await connectRelay({
  handler: {
    agentId: "my-agent",
    run: async function* (prompt) {
      yield { type: "status", content: "Working..." };
      yield { type: "done", content: "Result" };
    },
  },
});
```

## Exports

| Path               | Description                              |
| ------------------ | ---------------------------------------- |
| `relayo`           | All exports (server + client + protocol) |
| `relayo/server`    | WebSocket relay server                   |
| `relayo/client`    | Browser WebSocket client                 |
| `relayo/protocol`  | Shared types and constants               |

## License

MIT
