# Yu Shell patch on ureq 2.12.1

This branch is upstream ureq `2.12.1` (tag `2.12.1`, commit
`ba4555d54fd72e16fc737f7e8fcc812efb131260`, identical to the crates.io
archive with SHA-256 `02d1a66277ed75f640d608235660df48c8e3c19f3b4edb6a263315626cc3c01d`)
plus one commit that adds an optional `TcpConnector` hook. MIT and
Apache-2.0 licenses are unchanged.

It is published on crates.io as `swnb-ureq` (library name `ureq`) and
consumed only by the Yu handheld Shell (`yu-store-shell`) as
`ureq = { package = "swnb-ureq", version = "=2.12.1" }`, so builds behind
a crates.io mirror need no GitHub access. Other crates keep the upstream
`ureq` release; this is deliberately not a global `[patch.crates-io]`.

## What changes

Only three source files:

- `src/lib.rs` exports `TcpConnector`.
- `src/agent.rs` adds an optional connector to `AgentBuilder`, `None` by
  default, with an explicit setter.
- `src/stream.rs` calls the connector where upstream connects to each
  resolved address, and lets plain HTTP use an already boxed stream.

With no connector set, upstream TCP, TLS, proxy, parser, redirect and
deadline behavior is unchanged. The HTTP wrapper boxes the same socket by
default and copies no bytes; HTTPS keeps its TLS connector.

The Shell installs a cancellable connector only on its remote download
agent: the socket is registered with the operation before a nonblocking
Unix connect polled every 50 ms for cancellation. Rotation, refresh and
acknowledgement agents do not use the hook.

## Updating

Never force-push or delete a published commit. To move to another
upstream release, re-apply the three-file change on that tag in
a new branch, then re-run real pending-TCP, HTTP/TLS header/body/chunk,
certificate, redirect and proxy checks, the full Shell suite and the
production build before switching the pin. A green build alone is not
enough.
