# dnsdist 2.1.3 — OPNsense / FreeBSD 15 build

## Goal

Build, package, and validate upstream dnsdist 2.1.3 for the existing OPNsense FreeBSD 15 amd64 installation, without altering the running dnsdist 2.1.2 service until a tested replacement and rollback package are ready.

## Known baseline

- Active dnsdist version: 2.1.2 (Lua 5.4.8).
- Existing release package: v2.1.2-opnsense-freebsd15-amd64, reusable package SHA256 `09955ca8409146b1f440365ec43c6434b4802a02d321ec9ec37f65172b6d20a2`.
- Existing package is a **repack of an original FreeBSD binary**, not a locally rebuilt executable.
- Local changes documented in REPACK.md: map FreeBSD LMDB dependency to OPNsense lmdb; disable YAML sample auto-copy/removal; preserve original binary and other package metadata.
- Active service starts dnsdist using a Lua configuration. Do not commit production configurations, credentials, network-specific addresses, keys, or backup archives.
- Configuration and Technitium backup dated 2026-10-25 has a matching recorded SHA256.
- The active executable links to OPNsense-provided liblmdb.so.0, OpenSSL 3.5, Lua 5.4, GnuTLS, nghttp2, quiche, RE2, libsodium, and tinycdb.

## Source provenance

- Downloaded: `https://downloads.powerdns.com/releases/dnsdist-2.1.3.tar.xz`.
- Signature: `dnsdist-2.1.3.tar.xz.asc`.
- GPG reported a cryptographically valid signature by key `D6300CABCBF469BBE392E503A208ED4F8AF58446` (Rémi Gacogne). Independently check key fingerprint with an authenticated upstream source before treating signer identity as verified.
- Source tar archive passed `xz -t`.
- Source extracted to `/root/dnsdist-2.1.3-build/dnsdist-2.1.3` on the firewall.

## Build prerequisites and constraints

- Present: clang, make, gmake, autoconf, automake, libtool, pkgconf, Python/PyYAML, boost-libs, GnuTLS, LMDB, libsodium, quiche, RE2, Lua, nghttp2, tinycdb.
- Absent at initial inspection: rustc, cargo, bison, ragel. Do not assume all are mandatory for the release tarball.
- YAML support needs Rust/Cargo; native system package repositories did not report Rust/Cargo available.
- Prioritize matching the 2.1.2 feature set, including YAML, DNS-over-HTTP3 and DNS-over-QUIC. Check configure help and logs before deciding any flags.
- Do not enable FreeBSD package repositories on OPNsense or bulk-upgrade dependencies.
- Compiling on the production firewall requires resource limits and no interruption of the running service.

## Controlled procedure

1. Verify complete build prerequisites, source signer fingerprint and supported configure flags.
2. Configure and compile out-of-tree / into a staging directory; do **not** run `make install` over the active binary.
3. Verify the staged executable with `--version`, loader library resolution and compatibility checks.
4. Prepare a local .pkg with correct ABI, dependency names, sample-file lifecycle and manifest annotations; document any differences from 2.1.2.
5. Preserve backup and original 2.1.2 package; review upgrade/rollback commands.
6. Install only in a controlled maintenance window, then verify process, configured listeners and DNS resolution through the backend.
7. Publish the tested artifact with manifest, SHA256SUMS, build instructions and provenance. Only then merge verified changes to main.

## Status

- GitHub working branch: `dnsdist-2.1.3`.
- Preparation underway; **no 2.1.3 build, installation or test completed yet**.
