---
summary: "Source map for the Gateway codebase"
read_when:
  - You are navigating Gateway code
  - You want the main entry points for Gateway modules
  - You need to trace Gateway requests end to end
title: "Gateway code map"
---

# Gateway code map

Last updated: 2026-02-01

## Scope

The Gateway is the long running WebSocket and HTTP service that owns messaging
connections and the control plane. The core implementation lives under
`src/gateway` and is started by the
`openclaw gateway` CLI.

If you are looking for connector code, see [Connector code map](/channels/connectors-code-map).

## Primary entry points

- **CLI**: `src/cli/gateway-cli/run.ts` and `src/cli/gateway-cli/run-loop.ts` parse
  flags, resolve the target port, and call `startGatewayServer`.
- **Server bootstrap**: `src/gateway/server.impl.ts` owns `startGatewayServer` and
  wires configuration, plugins, runtime state, and shutdown handling.
- **Exports**: `src/gateway/server.ts` re-exports the Gateway server types and
  start helper for use in the rest of the codebase.

## Startup flow

1. `startGatewayServer` reads config (`src/config/config.ts`) and applies
   migrations.
2. It loads plugins and channel docks, then builds runtime state in
   `src/gateway/server-runtime-state.ts`.
3. Sidecars such as the browser control server, Gmail watcher, and internal
   hooks are started in `src/gateway/server-startup.ts`.
4. Channel connectors are started through `createChannelManager` in
   `src/gateway/server-channels.ts`.
5. The WebSocket server is attached via `src/gateway/server-ws-runtime.ts` and
   HTTP endpoints are attached via `src/gateway/server-http.ts`.

## Request handling

- **Authorization and dispatch** live in `src/gateway/server-methods.ts`.
- **Per domain handlers** are organized in `src/gateway/server-methods/*` and
  listed in `src/gateway/server-methods-list.ts`.
- **Protocol types** come from `src/gateway/protocol` and are generated from
  TypeBox schemas.

## HTTP surfaces

- **Control UI and static assets**: `src/gateway/server-http.ts` and
  `src/gateway/control-ui.ts`.
- **OpenAI compatible API**: `src/gateway/openai-http.ts`.
- **OpenResponses API**: `src/gateway/openresponses-http.ts`.
- **Tools Invoke API**: `src/gateway/tools-invoke-http.ts`.

## Runtime state and events

- **Presence and health**: `src/gateway/server/health-state.ts` and
  `src/gateway/server-health.ts`.
- **Sessions**: `src/gateway/server-sessions.ts` plus helpers in
  `src/gateway/sessions-*`.
- **Nodes and pairing**: `src/gateway/node-registry.ts`,
  `src/gateway/server-node-subscriptions.ts`, and
  `src/gateway/server-mobile-nodes.ts`.
- **Cron and maintenance**: `src/gateway/server-cron.ts` and
  `src/gateway/server-maintenance.ts`.

## Tests

Gateway tests are co located under `src/gateway` and end to end coverage lives
in files named `*.e2e.test.ts` alongside the server modules.

## Related docs

- [Gateway runbook](/gateway)
- [Gateway architecture](/concepts/architecture)
- [Gateway protocol](/gateway/protocol)
