---
title: CEP-25 Stateless Session Discovery and Capability Learning
description: Informational behavior model for first-message discovery tag negotiation in ContextVM sessions
---

# Stateless Session Discovery and Capability Learning

**Status:** Draft
**Author:** @contextvm-org
**Type:** Informational

## Abstract

This CEP documents the behavior model used for discovery and capability learning in stateless or handshake-light ContextVM sessions.

Instead of requiring prior public discovery or a full initialize/initialized roundtrip, peers can negotiate discovery information during the first message exchange of a session. The model is role-oriented: servers usually advertise metadata and transport capability tags, while clients advertise transport and negotiation capability tags.

This CEP is informational. It does not introduce new tags or event kinds. It clarifies how existing Standards Track CEPs are applied together in running implementations.

## Motivation

Discovery and capability negotiation behavior currently appears across multiple CEPs and SDK code paths. This can create ambiguity around:

- what "first response" means in practice,
- which tags are replayed by role,
- which parts are protocol-level versus SDK profile choices, and
- how unknown tags should be handled for extensibility.

A single informational behavior model reduces ambiguity while keeping Standards Track CEPs focused on their normative scope.

## Scope

This CEP describes:

- first-message exchange semantics for session bootstrap,
- role-oriented tag surfaces (server metadata and client capabilities),
- known-tag accessor expectations and raw-tag extensibility expectations, and
- implementation-profile boundaries for SDK-specific publication heuristics.

This CEP does not define:

- new event kinds,
- new discovery tag names,
- new payment semantics, or
- new relay selection rules at the protocol level.

## Terminology

- **Session**: Communication context between one client pubkey and one server pubkey for direct ContextVM traffic.
- **First message exchange**: First outbound message from each peer in a session (request or notification from client, response or notification from server), forming the first round trip.
- **Discovery tags**: Tags that communicate discoverability metadata or transport capability signals.
- **Known protocol tags**: Tags with explicit semantics in Standards Track CEPs.
- **Raw tags**: Uninterpreted tag tuples exposed for extension protocols.

## Behavior Model

### First-Message Exchange Negotiation

Recommended interoperable behavior is:

1. The client includes its discovery or negotiation tags on its first outbound message in the session.
2. The server includes its discovery tags on its first outbound message in the session.
3. After the first round trip, both peers MAY stop replaying unchanged discovery tags for that session.

This keeps negotiation available even when no full initialize handshake is observed by one side.

### Role-Oriented Tag Surfaces

Server-side discovery surface usually includes:

- server metadata tags (for example `name`, `about`, `website`, `picture`),
- transport capability tags defined by Standards Track CEPs, and
- optional extension tags.

Client-side discovery and negotiation surface usually includes:

- transport capability tags (for example encryption and oversized support),
- payment negotiation tags where applicable (see CEP-8), and
- optional extension tags.

### Relationship to Initialize

`initialize` and `notifications/initialized` remain valid and recommended lifecycle signals.

This behavior model does not replace initialize semantics. It ensures capability learning can still converge when sessions operate statelessly or skip full handshake paths.

## Tag Handling and Extensibility

### Known Tags

Implementations SHOULD provide convenience accessors for known protocol tags so applications do not need to parse raw tuples manually.

### Raw Tag Access

Implementations SHOULD also expose raw tags so custom protocols and private extensions can coexist with standard tags.

### Unknown Tag Preservation

For interoperability and extension safety, clients and servers SHOULD preserve unknown discovery tags as raw tuples and avoid discarding them solely because they are not currently interpreted.

When merging discovery observations across multiple inbound events, tuple-level deduplication is RECOMMENDED so unknown tags remain stable and inspectable.

## Standards Mapping

- **CEP-6**: Canonical public announcement and discovery surface and tag vocabulary.
- **CEP-19**: Semantics of `support_encryption_ephemeral` and wrap-kind negotiation.
- **CEP-8**: Payment negotiation tags and stateless PMI behavior.
- **CEP-17**: Relay list metadata for discoverability.

This CEP explains how these surfaces are learned during direct session bootstrap. It does not redefine their normative meaning.

## Implementation Profile Boundary

Some operational behaviors are implementation profile choices, not protocol requirements.

Example (TypeScript SDK profile):

- skipping default public bootstrap publication when all operational relays are local-only unless bootstrap relays are explicitly configured, and
- falling back to direct publish path when discoverability targets are not websocket URLs.

These choices improve determinism in local and test environments and should not be interpreted as mandatory behavior for all ContextVM implementations.

## Reference Implementation

A reference behavior implementation is available in the ContextVM SDK transport layer:

- https://github.com/ContextVM/sdk/blob/master/src/transport/nostr-client-transport.ts
- https://github.com/ContextVM/sdk/blob/master/src/transport/nostr-server-transport.ts
- https://github.com/ContextVM/sdk/blob/master/src/transport/discovery-tags.ts

## Dependencies

- [CEP-6: Public Server Announcements](/spec/ceps/cep-6)
- [CEP-8: Capability Pricing and Payment Flow](/spec/ceps/cep-8)
- [CEP-17: Server Relay List Metadata](/spec/ceps/cep-17)
- [CEP-19: Ephemeral Gift Wraps](/spec/ceps/cep-19)
