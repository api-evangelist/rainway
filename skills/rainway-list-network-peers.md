---
name: List Rainway network peers
description: Enumerate and count the active Rainway peers connected to the network for an organization, for building multi-peer connection UIs.
api: openapi/rainway-hub-api-openapi.yml
operations: [getPeers, getPeersCount]
---

# List Rainway network peers

Use the Rainway Hub API to discover which peers (devices/hosted apps) are currently
connected to the Rainway Network under your organization, so a client can present a
list of connectable applications.

## Authentication
- HTTP Basic auth with a Rainway API Key Pair. Username = public key (`pk_live_...`),
  password = secret key (`sk_live_...`). Generate keys in the Rainway Hub
  (https://hub.rainway.com/keys).
- Base URL: `https://hub-api.rainway.com/v0`
- Your `orgId` is shown in the URL bar when logged into the Rainway Hub.

## Steps
1. **Count peers** — call `getPeersCount` (`GET /orgs/{orgId}/peers/count`) to get
   `body.value.totalItems`. Use this to decide whether to page.
2. **List peers** — call `getPeers` (`GET /orgs/{orgId}/peers`) with `limit` (0-100,
   default 100) and optional `cursor`. Read the peer array at `body.value.peers`.
3. **Page** — if `body.value.metadata.next` is present (or `totalItems` exceeds what you
   have), repeat `getPeers` using the returned `metadata.cursor` until exhausted.
4. **Connect** — each `Peer.id` carries a trailing `n`; trim the last character to get a
   connectable ID (BigInt in JavaScript). Match `externalId` to tie a peer back to a user.

## Conventions & errors
- Responses are enveloped: `{ metadata, body: { value: { metadata, peers } } }`.
- Pass an arbitrary `context` string to have it echoed back in `metadata.context` for
  request/response correlation.
- Errors return `{ metadata, traceId, error: { code, description } }` — log `traceId`
  for support. See `errors/rainway-problem-types.yml` and `conventions/rainway-conventions.yml`.
- Both operations are read-only and safe to retry.
