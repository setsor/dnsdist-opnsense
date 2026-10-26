# Installation and rollback — dnsdist 2.1.3

Instructions for the current **unofficial dnsdist 2.1.3 OPNsense / FreeBSD 15 amd64** package. This build was tested on OPNsense / FreeBSD **15.1-RELEASE-p4 amd64**, not on every FreeBSD 15 installation.

Download the package, `manifest.json` and `SHA256SUMS` from the [dnsdist 2.1.3 release](https://github.com/setsor/dnsdist-opnsense/releases/tag/v2.1.3-opnsense-freebsd15-amd64).

For the previous 2.1.2 repack, see [historical installation instructions](INSTALL-2.1.2.md).

> **Warning:** dnsdist is a DNS infrastructure service. Ensure administrative access by IP address or console, an independent configuration backup, and a tested prior package for rollback before changing it. Neither PowerDNS nor OPNsense endorses this package.

## 1. Verify the target and the download

This package targets `FreeBSD:15:amd64`, with runtime linkage to the **base-system OpenSSL ABI 35** (`libssl.so.35`, `libcrypto.so.35`). Do not force-install on a different ABI or with incompatible libraries.

```sh
freebsd-version
uname -m
pkg -vv | grep -E '^(ABI|ALTABI)'
```

From the directory containing all three release assets:

```sh
sha256sum -c SHA256SUMS
pkg info -F ./dnsdist-2.1.3-opnsense-freebsd15-amd64.pkg
```

The expected SHA256 of the **package** is:

```text
44ff2addaa44dfbe094628e7a3c16eb75991b88f136b34834c29e4902dae11c2
```

The package records 11 package dependencies, including `abseil`, `fstrm`, `quiche`, `lmdb`, `lua54` and other shared libraries. Make sure the required libraries are available on the target; do not enable a foreign FreeBSD package repository on OPNsense just to resolve them.

## 2. Back up the current installation

```sh
dnsdist --version
pkg query '%n %v %o' dnsdist
service dnsdist status

stamp=$(date +%Y%m%d-%H%M%S)
mkdir -p "/root/dnsdist-backup-${stamp}"
pkg create -o "/root/dnsdist-backup-${stamp}" dnsdist
tar -C /usr/local/etc -czf "/root/dnsdist-config-${stamp}.tar.gz" dnsdist
sha256 /usr/local/etc/dnsdist/dnsdist.conf
```

Store the previous package and configuration checksum securely. Back up separately any certificates, other configuration directories (including `dnsdist-shadow` if in use), or referenced files outside `/usr/local/etc/dnsdist`.

The previously published [2.1.2 package](https://github.com/setsor/dnsdist-opnsense/releases/tag/v2.1.2-opnsense-freebsd15-amd64) has SHA256:

```text
09955ca8409146b1f440365ec43c6434b4802a02d321ec9ec37f65172b6d20a2
```

## 3. Install or replace from the local archive

Perform the following with console/IP management access and a planned short DNS outage. These commands apply to the **locally downloaded** release asset.

```sh
service dnsdist stop
pkg add -f ./dnsdist-2.1.3-opnsense-freebsd15-amd64.pkg
dnsdist --check-config -C /usr/local/etc/dnsdist/dnsdist.conf
service dnsdist start
```

Do not use `pkg delete dnsdist` as an intermediate step. FreeBSD `pkg add -f` was the successfully tested replacement operation on the original installation.

The package retains the `_dnsdist` service user/group, retains existing modified `dnsdist.conf` via the package configuration lifecycle, and does not auto-create a live YAML configuration from `dnsdist.yml.sample`. A separate configuration backup remains mandatory.

## 4. Verify the service and DNS

```sh
pkg query '%n %v %o' dnsdist
dnsdist --version
service dnsdist status
sockstat -4 -6 -l | grep dnsdist
drill @127.0.0.1 example.com A
sha256 /usr/local/etc/dnsdist/dnsdist.conf
pkg check -d -n dnsdist
```

Confirm `dnsdist 2.1.3`, the actual configured listeners, successful DNS responses, no dependency errors and an unchanged configuration hash. Test any LAN, IPv6, DNS-over-TLS, DNS-over-HTTPS, DNS-over-QUIC, HTTP/3 or DNSCrypt listeners that **your own deployment actually uses**.

On the tested system, DNS A lookups returned `NOERROR` on loopback and LAN, and the configuration hash was unchanged. There has been **no comprehensive live regression test** of encrypted transports or sustained load.

## 5. Roll back if needed

Stop the new service, reinstall the **previous known-good local package**, restore configuration if required, then validate service health:

```sh
service dnsdist stop
pkg add -f /path/to/previous/dnsdist-2.1.2.pkg
# Restore the backed-up dnsdist.conf only if it was changed or removed.
service dnsdist start
dnsdist --version
service dnsdist status
drill @127.0.0.1 example.com A
```

Replace the example package path with your actual verified rollback file. If the service cannot be restored remotely, use console access.

## Build provenance

See [BUILD-2.1.3.md](BUILD-2.1.3.md) for source validation, compiler options, the exact OpenSSL relink step, package dependencies, installation outcome, and validation boundaries. The package is unofficial and is **not** guaranteed to be compatible with future OPNsense/FreeBSD updates without revalidation.
