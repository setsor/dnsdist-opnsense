# Source and build provenance

This file records the provenance embedded in the dnsdist 2.1.2 package manifest.

## Upstream identity

```text
Package:      dnsdist
Version:      2.1.2
Origin:       dns/dnsdist
WWW:          https://dnsdist.org/
Architecture: FreeBSD:15:amd64
Arch:         freebsd:15:x86:64
Prefix:       /usr/local
```

Upstream PowerDNS source repository:

https://github.com/PowerDNS/pdns

This repository does not claim authorship of dnsdist.

## FreeBSD build provenance

The manifest records:

```text
FreeBSD_version:            1500068
build_timestamp:            2026-09-10T10:38:51+0000
built_by:                   poudriere-git-3.4.8-2-g2e891d4a
port_checkout_unclean:      no
port_git_hash:              ad7aa83df1ee50ed8d35dff5198e80e31ef06faa
ports_top_checkout_unclean: no
ports_top_git_hash:         e26b1e4bc8ec94e73586539808ef21695232e8aa
```

The package CPE annotation is:

```text
cpe:2.3:a:powerdns:dnsdist:2.1.2:::::freebsd15:x64
```

## Local repack provenance

The package manifest contains:

```text
local_repack: Original 2.1.2 binary; legacy file metadata; LMDB 0.9.35 dependency mapped to OPNsense lmdb; YAML sample auto-copy/removal disabled
```

This is the authoritative summary of the local package adaptation.

## Package checksum

```text
SHA256 (dnsdist-2.1.2.pkg) = 09955ca8409146b1f440365ec43c6434b4802a02d321ec9ec37f65172b6d20a2
```

When the package is published as a GitHub Release asset with a more descriptive filename, the file bytes are expected to remain unchanged and therefore retain the same SHA256.

## Package licenses

The package metadata records:

- GPLv2
- ISC
- MIT

The package contains corresponding license material under:

```text
/usr/local/share/licenses/dnsdist-2.1.2/
```

including `GPLv2`, `ISCL`, `MIT`, `LICENSE`, and `catalog.mk`.

## Package users and groups

The package manifest declares:

```text
user:  _dnsdist
group: _dnsdist
```

Its installation script creates the `_dnsdist` user and group with UID/GID 208 when they do not already exist.

## Reproducibility boundary

The information above allows a reviewer to identify the package version, FreeBSD port revision, ports-tree revision, builder version, build timestamp, ABI, dependencies and local repack annotation.

This repository does not claim bit-for-bit reproducibility unless a future release explicitly provides the complete build environment and demonstrates a reproducible rebuild.
