---
summary: "Source map for channel connectors and plugin architecture"
read_when:
  - You are navigating channel connector code
  - You need to understand where a channel plugin lives
  - You are adding a new connector or debugging one
title: "Connector code map"
---

# Connector code map

Last updated: 2026-02-01

## Scope

Connectors ("channels") are the integrations that bring chat platforms into the
Gateway. Core channel wiring lives in `src/channels` and the individual
connectors are implemented as plugins under `extensions/*`. The Gateway loads
all configured channel plugins at startup.

If you are looking for the Gateway overview, see [Gateway code map](/gateway/code-map).

## Core channel wiring

- **Registry**: `src/channels/registry.ts` defines the core channel IDs, labels,
  and docs metadata.
- **Lightweight dock**: `src/channels/dock.ts` exposes shared capabilities,
  mention handling, threading helpers, and allowlist parsing without loading the
  full plugin runtime.
- **Plugin API types**: `src/channels/plugins/types.ts` and
  `src/channels/plugins/types.*` define the adapter interfaces used by each
  connector.
- **Plugin SDK**: `src/plugin-sdk/index.ts` re-exports the connector types for
  extension packages.

## Plugin lifecycle

1. Plugin entry points live in `extensions/<channel>/index.ts` and call
   `api.registerChannel({ plugin })`.
2. The Gateway loads plugins in `src/plugins/loader.ts` and stores them in the
   runtime registry (`src/plugins/runtime`).
3. `listChannelPlugins` in `src/channels/plugins/index.ts` enumerates the
   registered plugins.
4. `createChannelManager` in `src/gateway/server-channels.ts` starts each
   connector and tracks its runtime status.

## Connector layout (per channel)

Most connector packages follow this pattern (example: Telegram):

- `extensions/telegram/index.ts`: plugin registration entry point.
- `extensions/telegram/src/channel.ts`: plugin definition (capabilities,
  configuration, adapter hooks, security, directory helpers, etc).
- `extensions/telegram/src/runtime.ts`: shared runtime state for the connector.

Other connectors use the same structure under their `extensions/<channel>`
folders (WhatsApp, Discord, Slack, Signal, iMessage, and plugin-only channels
like Matrix or Zalo).

## Shared channel behaviors

- **Mention gating + tool policy**: `src/channels/plugins/group-mentions.ts`.
- **Allowlists + routing**: `src/channels/allowlists` and
  `src/channels/plugins/channel-config.ts`.
- **Typing indicators + reactions**: `src/channels/typing.ts` and
  `src/channels/ack-reactions.ts`.

## Tests

Connector tests live alongside their implementation (either in `extensions/*`
for plugin channels or `src/channels` for shared behaviors). Look for
`*.test.ts` files next to the module you are touching.

## Related docs

- [Channels overview](/channels)
- [Gateway architecture](/concepts/architecture)
- [Gateway runbook](/gateway)
