# dnsdist for OPNsense / FreeBSD 15

This repository publishes **unofficial compatibility repacks of dnsdist for OPNsense / FreeBSD 15 amd64**.

The current package is based on **dnsdist 2.1.2** from the FreeBSD port `dns/dnsdist`. The package manifest records the target ABI as `FreeBSD:15:amd64` / `freebsd:15:x86:64`.

> [!IMPORTANT]
> This repository is not affiliated with, endorsed by, or an official distribution channel for PowerDNS or OPNsense.
>
> The dnsdist binary is not presented as a custom PowerDNS build. The package manifest records the repack as using the **original dnsdist 2.1.2 binary**, with compatibility changes limited to package metadata and package lifecycle behavior described below.

## Current package

| Item | Value |
| --- | --- |
| dnsdist version | 2.1.2 |
| FreeBSD ABI | `FreeBSD:15:amd64` |
| FreeBSD arch | `freebsd:15:x86:64` |
| Package origin | `dns/dnsdist` |
| Package SHA256 | `09955ca8409146b1f440365ec43c6434b4802a02d321ec9ec37f65172b6d20a2` |
| FreeBSD version recorded at build | `1500068` |
| Build timestamp | `2026-09-10T10:38:51+0000` |
| Builder | `poudriere-git-3.4.8-2-g2e891d4a` |
| FreeBSD port commit | `ad7aa83df1ee50ed8d35dff5198e80e31ef06faa` |
| FreeBSD ports tree commit | `e26b1e4bc8ec94e73586539808ef21695232e8aa` |

## Why this repack exists

The package was adapted for the tested OPNsense / FreeBSD 15 environment without changing the dnsdist 2.1.2 executable.

The package manifest records these local changes:

1. Preserve the original dnsdist 2.1.2 binary.
2. Preserve legacy file metadata from the source package.
3. Map the LMDB dependency to the OPNsense package `lmdb 0.9.35,1`.
4. Disable automatic copy/removal behavior for `dnsdist.yml.sample`.

The normal `dnsdist.conf.sample` package lifecycle script remains present.

See [REPACK.md](REPACK.md) for the exact packaging changes and rationale.

## Build options recorded in the package

Enabled:

- CDB
- GNUTLS
- IPCIPHER
- LMDB
- LUA
- OPENSSL

Disabled:

- DNSTAP
- LUAJIT
- SNMP

The package links against, among others, OpenSSL 3.5 (`libssl.so.35` / `libcrypto.so.35`), GnuTLS, LMDB, Lua 5.4, nghttp2, quiche, RE2, libsodium and tinycdb.

## Documentation

- [INSTALL.md](INSTALL.md) — verification, backup, installation, validation and rollback
- [REPACK.md](REPACK.md) — what was changed and what was deliberately not changed
- [SOURCE-PROVENANCE.md](SOURCE-PROVENANCE.md) — package, port and build provenance

## Release asset

The GitHub Release will publish the package with a descriptive asset name:

`dnsdist-2.1.2-opnsense-freebsd15-amd64.pkg`

The internal FreeBSD package identity remains:

`dnsdist-2.1.2`

Expected SHA256:

```text
09955ca8409146b1f440365ec43c6434b4802a02d321ec9ec37f65172b6d20a2
```

Always verify the checksum before installation.

## Upstream

- dnsdist documentation: https://dnsdist.org/
- PowerDNS source repository: https://github.com/PowerDNS/pdns
- FreeBSD package origin: `dns/dnsdist`

dnsdist and the files contained in the package remain subject to their respective upstream licenses. The package metadata records **GPLv2, ISC and MIT** licenses and includes the corresponding license files.

## Scope

This repository documents and distributes the tested OPNsense / FreeBSD 15 package adaptation. It is not intended to replace upstream dnsdist, the FreeBSD ports tree, or OPNsense package management.
