# dnsdist for OPNsense / FreeBSD 15

This repository documents and publishes **unofficial dnsdist builds and compatibility repacks for OPNsense / FreeBSD 15 amd64**.

The currently published release is a **dnsdist 2.1.2 compatibility repack** from the FreeBSD port `dns/dnsdist`. A separately compiled **dnsdist 2.1.3** package has been validated on OPNsense and published as a GitHub Release.

> [!IMPORTANT]
> This repository is not affiliated with, endorsed by, or an official distribution channel for PowerDNS or OPNsense.
>
> Version **2.1.2** retains the original FreeBSD executable with package-metadata compatibility changes. Version **2.1.3** is an independently compiled source build; its documented provenance and testing differ from the 2.1.2 repack.

## Latest release — dnsdist 2.1.3

The local package was successfully installed and validated on OPNsense / FreeBSD 15.1 amd64 on 2026-10-26. It preserves the system OpenSSL ABI 35, enables YAML, DNSCrypt, DoT, DoH, DoQ, DoH3, fstrm and other features, and passed configuration, listener, basic DNS resolution and package dependency checks. Full encrypted-protocol testing has **not** been completed.

Build procedure and exact package checksum: [BUILD-2.1.3.md](BUILD-2.1.3.md).

Download the published [dnsdist 2.1.3 package, manifest and checksums](https://github.com/setsor/dnsdist-opnsense/releases/tag/v2.1.3-opnsense-freebsd15-amd64). Its package SHA256 is `44ff2addaa44dfbe094628e7a3c16eb75991b88f136b34834c29e4902dae11c2`. Verify downloads before installation. See also [release notes](RELEASE-NOTES-2.1.3.md).

## Current published release (2.1.2)

**[v2.1.2-opnsense-freebsd15-amd64](https://github.com/setsor/dnsdist-opnsense/releases/tag/v2.1.2-opnsense-freebsd15-amd64)**

Release assets:

- `dnsdist-2.1.2-opnsense-freebsd15-amd64.pkg`
- `manifest.json`
- `SHA256SUMS`

## Published 2.1.2 package

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

## Package checksum

The package asset is:

`dnsdist-2.1.2-opnsense-freebsd15-amd64.pkg`

The internal FreeBSD package identity remains:

`dnsdist-2.1.2`

SHA256:

```text
09955ca8409146b1f440365ec43c6434b4802a02d321ec9ec37f65172b6d20a2
```

GitHub independently reports the same SHA256 digest for the published release asset.

Always verify the checksum before installation.

## Upstream

- dnsdist documentation: https://dnsdist.org/
- PowerDNS source repository: https://github.com/PowerDNS/pdns
- FreeBSD package origin: `dns/dnsdist`

dnsdist and the files contained in the package remain subject to their respective upstream licenses. The package metadata records **GPLv2, ISC and MIT** licenses and includes the corresponding license files.

## Scope

This repository documents and distributes the tested OPNsense / FreeBSD 15 package adaptation. It is not intended to replace upstream dnsdist, the FreeBSD ports tree, or OPNsense package management.
