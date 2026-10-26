# dnsdist for OPNsense / FreeBSD 15

Unofficial dnsdist builds and compatibility packages for OPNsense / FreeBSD 15 amd64.

> **Not an official PowerDNS or OPNsense distribution.** Validate the target ABI, installed shared libraries and configuration before changing a production DNS service. Back up the previous package and configuration.

## Latest release — dnsdist 2.1.3

**[Download dnsdist 2.1.3 for OPNsense / FreeBSD 15 amd64](https://github.com/setsor/dnsdist-opnsense/releases/tag/v2.1.3-opnsense-freebsd15-amd64)**

The 2.1.3 package was compiled from upstream source on OPNsense / FreeBSD **15.1-RELEASE-p4 amd64** and installed and operationally validated on **2026-10-26**. Unlike the earlier 2.1.2 release, it is a **local source build**, not a repack of an existing FreeBSD executable.

| Item | dnsdist 2.1.3 |
| --- | --- |
| Package asset | `dnsdist-2.1.3-opnsense-freebsd15-amd64.pkg` |
| SHA256 | `44ff2addaa44dfbe094628e7a3c16eb75991b88f136b34834c29e4902dae11c2` |
| Platform tested | OPNsense / FreeBSD 15.1 amd64 |
| OpenSSL dynamic libraries | Base-system `libssl.so.35` and `libcrypto.so.35` |
| Enabled features | Lua 5.4, YAML, DNSCrypt, DoT, DoH, DoQ, DoH3, fstrm/DNSTAP, LMDB, RE2, GnuTLS, OpenSSL and others |
| Verified | Service startup, configuration check, UDP/TCP listeners, basic DNS resolution, dependency checks and unchanged configuration |

The release includes the package, `manifest.json` and `SHA256SUMS`. Verify the downloaded files before installation. The build includes encrypted-DNS protocol support, but comprehensive live testing of all encrypted frontends and sustained load **has not been performed**.

**Current documentation:**

- [Install, verify and roll back dnsdist 2.1.3](INSTALL.md)
- [Build and package provenance, OpenSSL fix and validation](BUILD-2.1.3.md)
- [Release notes](RELEASE-NOTES-2.1.3.md)

## Previous release — dnsdist 2.1.2

**[dnsdist 2.1.2 OPNsense compatibility repack](https://github.com/setsor/dnsdist-opnsense/releases/tag/v2.1.2-opnsense-freebsd15-amd64)**

The previous release retains the **original FreeBSD 2.1.2 binary** and changes only package metadata and lifecycle handling for the tested OPNsense environment, including its LMDB dependency mapping and disabled YAML-sample auto-copy/removal.

Package SHA256: `09955ca8409146b1f440365ec43c6434b4802a02d321ec9ec37f65172b6d20a2`.

Historical documentation:

- [Installation and rollback for 2.1.2](INSTALL-2.1.2.md)
- [2.1.2 repack modifications](REPACK.md)
- [2.1.2 source, port and build provenance](SOURCE-PROVENANCE.md)

## Release source and maintenance

- Upstream project: [PowerDNS/pdns](https://github.com/PowerDNS/pdns)
- dnsdist documentation: [dnsdist.org](https://dnsdist.org/)
- FreeBSD package origin: `dns/dnsdist`

The **2.1.3 release tag** was created before the publication-status text was updated, so its historical README/BUILD snapshot can still say that the release is pending. The release assets are published and their package SHA256 has been verified against GitHub's asset digest. Consult this `main` branch for the corrected current documentation. Published tag history has not been rewritten.

Keep live `dnsdist.conf`, private network details, secrets and certificates out of GitHub.
