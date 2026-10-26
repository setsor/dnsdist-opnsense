# dnsdist 2.1.3 — OPNsense / FreeBSD 15 amd64

Unofficial, independently compiled dnsdist 2.1.3 package for OPNsense / FreeBSD 15.1 amd64.

## Key details

- Built locally from the upstream dnsdist 2.1.3 source tarball, with a valid detached GPG signature.
- Compiled with Lua 5.4, YAML support (Rust), DNSCrypt, DNS-over-TLS, DNS-over-HTTPS, DNS-over-QUIC, DNS-over-HTTP/3, GnuTLS, OpenSSL, IPCipher, IPCrypt2, LMDB, RE2, CDB, Protobuf and fstrm/DNSTAP.
- Explicitly linked to the OPNsense base OpenSSL 3.5 ABI libraries `libssl.so.35` and `libcrypto.so.35` at link time, rather than the ports OpenSSL ABI; these are dynamic shared libraries, not static linking.
- Stripped executables, FreeBSD `pkg` manifest with 11 package dependencies, and preserved `_dnsdist` service user and Lua configuration lifecycle.
- Built on OPNsense / FreeBSD 15.1-RELEASE-p4, amd64.
- Not an official PowerDNS or OPNsense release.

## Operational test

Successfully installed on 2026-10-26 with `pkg add -f`. Verified `dnsdist --version`, `--check-config`, service startup, configured UDP/TCP listeners, DNS resolution via localhost and LAN, unchanged configuration checksum, and `pkg check -d -n dnsdist`.

This validation does not constitute comprehensive live testing of the TLS/HTTP/QUIC transports, DNSCrypt, or sustained production load.

## Release assets

- `dnsdist-2.1.3-opnsense-freebsd15-amd64.pkg`
- `manifest.json`
- `SHA256SUMS`

Package SHA256:
```text
44ff2addaa44dfbe094628e7a3c16eb75991b88f136b34834c29e4902dae11c2
```

Verify downloads using `sha256sum -c SHA256SUMS` from the asset directory.

## Upgrade

**Back up the existing package and configuration before proceeding.** DNS service will be interrupted briefly. Ensure a direct IP/console management route is available.

```sh
service dnsdist stop
pkg add -f ./dnsdist-2.1.3-opnsense-freebsd15-amd64.pkg
dnsdist --check-config -C /usr/local/etc/dnsdist/dnsdist.conf
service dnsdist start
dnsdist --version
service dnsdist status
```

Revalidate local name resolution and the actual deployment listeners after restarting.

## Documentation and rollback

See [BUILD-2.1.3.md](https://github.com/setsor/dnsdist-opnsense/blob/main/BUILD-2.1.3.md) for source provenance, build procedure, exact OpenSSL relink method, package metadata, test scope and rollback notes.

The previous published [dnsdist 2.1.2 release](https://github.com/setsor/dnsdist-opnsense/releases/tag/v2.1.2-opnsense-freebsd15-amd64) remains available for rollback with an independent configuration backup.

**Do not include live production configuration files or private certificates in GitHub assets.**
