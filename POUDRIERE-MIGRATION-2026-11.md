# Poudriere build and OPNsense migration — dnsdist 2.1.3 (2026-11-17/18)

This is a **separate build and deployment record** for the November 2026 rebuild. It supplements, but does not overwrite, the **October 26** source-built release described in [BUILD-2.1.3.md](BUILD-2.1.3.md), [INSTALL.md](INSTALL.md), and the existing GitHub release assets.

**Status:** the Poudriere build succeeded; the selected packages were installed on the OPNsense 26.7 / FreeBSD 15.1 amd64 target on November 18; configuration validation, dynamic-library resolution, DNS UDP/TCP checks, LAN/VLAN lookups, and package dependency checks passed. Comprehensive encrypted-protocol, client-IP/PROXY-protocol, and load testing were **not** performed.

This is an unofficial integration record, not an upstream PowerDNS source patch or an official OPNsense package release. No production configuration, keys, certificates, private network addresses, or system backups are stored in this repository.

## What changed from the October 2026 build

| Aspect | Original October build | November Poudriere build |
| --- | --- | --- |
| dnsdist | 2.1.3 | 2.1.3 (unchanged) |
| Build environment | Locally on OPNsense, Autoconf / GNU make | Isolated FreeBSD 15.1 amd64 Parallels VM, Poudriere 3.4.8, FreeBSD Ports / Meson |
| OpenSSL | Base system `libssl.so.35` / `libcrypto.so.35`, manually corrected linker invocation | Base system ABI 35 via `DEFAULT_VERSIONS+= ssl=base`, verified in ELF |
| LMDB dependency | `lmdb-0.9.35,1`, SONAME `liblmdb.so.0` | `lmdb0-0.9.36_1`, SONAME `liblmdb.so.0` |
| Quiche | `0.24.5_7` | `0.24.5_10` |
| libnghttp2 | `1.70.0` | `1.71.0` |
| Lua | 5.4.8 | 5.4.8 |
| GnuTLS | 3.8.13 | 3.8.13 |
| Package configuration | `LMDB=on`, built against OPNsense's installed `lmdb` | `LMDB=on`, `LMDB0=on`, via official FreeBSD port options |
| GitHub release asset | October 2.1.3 package, SHA256 below | New package created and deployed locally; **not claimed to be published as a GitHub release** |

The October release package and the November Poudriere package are **not interchangeable artifacts**: they have different package manifests, dependencies, build provenance, and SHA256 digests. The runtime feature names reported by `dnsdist --version` matched, but that is not a full feature regression test.

## Poudriere environment and build configuration

- Builder OS/jail: FreeBSD **15.1-RELEASE-p6** amd64; jail `151amd64`.
- Ports tree: `default`, commit `9f17300fa336bed3505395257f88ea4e7ce4fd45`.
- Poudriere set: `opnsense_dns`; package repository `151amd64-default-opnsense_dns`.
- Poudriere options in `/usr/local/etc/poudriere.d/151amd64-default-opnsense_dns-options/dns_dnsdist/options`:

```text
DNSTAP=on
IPCIPHER=on
LMDB0=on
SNMP=off
LUA=on
LUAJIT=off
GNUTLS=on
OPENSSL=on
CDB=on
LMDB=on
```

- Poudriere make configuration contains:

```make
DEFAULT_VERSIONS+= ssl=base
```

- Poudriere `PACKAGE_FETCH_BLACKLIST="dnsdist"` ensures the dnsdist port is built locally rather than fetched from the public package repository.
- The large Rust toolchain was compiled from the `lang/rust` port, version **1.99.1_2**. It took approximately **10h29m** total Poudriere time. Dependency artifacts for the subsequent dnsdist build were reused.
- The final build was invoked with:

```sh
poudriere bulk -b latest -j 151amd64 -p default -z opnsense_dns dns/dnsdist
```

The final LMDB0 run built **dnsdist-2.1.3** in approximately **21m25s** package build time, fetched `lmdb0-0.9.36_1`, completed with **Built: 1, Failed: 0**, and committed its package repository. An earlier successful build without LMDB0 instead linked `liblmdb.so.1`, so was **not** selected for OPNsense deployment.

Upstream source was not modified as part of this reported rebuild. The integration-specific differences are the FreeBSD port build, its option set, selected dependency versions, and the operational migration. The production Lua configuration was preserved.

## Final package and verification

November Poudriere package: `dnsdist-2.1.3.pkg`, FreeBSD:15:amd64, approximately 3.4 MB.

```text
dnsdist-2.1.3.pkg
SHA256 76ae940d3cdb14c4fa135346bdd7b1753b3ce2df8281c5d8d2ecb6dbe7350249
```

Its ELF dependencies include the base-system `libssl.so.35` and `libcrypto.so.35`, `liblmdb.so.0`, `libquiche.so.0`, and `libnghttp2.so.14`. Package metadata declares `lmdb0-0.9.36_1`, `quiche-0.24.5_10` and `libnghttp2-1.71.0`.

On the target, the package checksum matched; `ldd` resolved all required shared libraries; the extracted executable reported dnsdist **2.1.3 (Lua 5.4.8)** with the same enabled-feature list as the installed October build; `--check-config` returned **Configuration OK**. Initial resolution tests preceded installation, and the final checks followed installation.

### Local artifacts (not published here)

| Archive | SHA256 |
| --- | --- |
| November Poudriere dnsdist 2.1.3 | `76ae940d3cdb14c4fa135346bdd7b1753b3ce2df8281c5d8d2ecb6dbe7350249` |
| Original October dnsdist 2.1.3 (rollback) | `44ff2addaa44dfbe094628e7a3c16eb75991b88f136b34834c29e4902dae11c2` |
| Quiche 0.24.5_10 | `ab2f51e53af0083ff541b008b263c3ef247f42fc4d2da2cb4676c9bd00279bd4` |
| libnghttp2 1.71.0 | `ffc7ceaf2acb07bd772072697e234a9bd99261759ab17d12d244992d7d9ca971` |
| LMDB 0.9.35 rollback, recreated with `pkg create` | `ab65b3e79c55d5beda8cb1628db4b34189f18c41c25960c9dc404f91c3b83fda` |
| Quiche 0.24.5_7 rollback, recreated with `pkg create` | `33314addefa58c93bf84d5a6f3c273b2a42603b123e3d23ec9f3f72820dab5a5` |
| libnghttp2 1.70.0 rollback, recreated with `pkg create` | `b8b2f13b11820576ad48e9097c270075eb3fe700f01f6bc66fffd9f14214acd8` |

The recreated rollback archives use the local installed files and package database; they are not asserted to be byte-identical to historical official archives.

## OPNsense deployment: isolated local package repository

The official OPNsense 26.7 repository offered `lmdb0-0.9.36_1`, but **did not list dnsdist or Quiche**, and its `libnghttp2` was **1.70.0**. The Quiche and dnsdist packages were built locally, and `libnghttp2` 1.71.0 was supplied from the Poudriere repository.

Direct `pkg install -n` against standalone archive paths did not choose all requested versions consistently. A *temporary*, local-only pkg repository containing exactly the four selected packages was created:

```text
dnsdist-2.1.3.pkg
quiche-0.24.5_10.pkg
libnghttp2-1.71.0.pkg
lmdb0-0.9.36_1.pkg
```

Its catalog was generated with `pkg repo`. A separate temporary repository configuration with `file://` URL, `signature_type: "none"`, and priority 100 was passed explicitly using `pkg -R`. **Do not expose this unsigned repository over the network or enable it globally.** It is a provenance-verified, local deployment mechanism, not a general package source.

The following command was first run with `-n` to inspect the transaction, then without `-n` to perform it:

```sh
pkg -R /path/to/temporary-repo-config install -r dnsdist-local \
  dnsdist quiche libnghttp2 lmdb0
```

The solver reported a file conflict on `liblmdb.so.0`, resolved it by **removing** the previous `lmdb-0.9.35,1`, and then planned:

1. Install `lmdb0-0.9.36_1`.
2. Upgrade `libnghttp2` 1.70.0 → 1.71.0.
3. Upgrade `quiche` 0.24.5_7 → 0.24.5_10.
4. Reinstall `dnsdist-2.1.3` (option `LMDB0` added).

The identical plan was observed in the final dry run before deployment; the actual package transaction completed successfully. The pre-existing `_dnsdist` user and group were reused, not deleted. `dnsdist.conf` remained in place.

LLVM 19.1.7_4 was already installed on OPNsense. Although the Quiche 0.24.5_10 package declares `llvm19-19.1.7_5`, the solver did not request a compiler upgrade and the installed system passed `pkg check -d -n`. Quiche's manifest did not list LLVM shared libraries as runtime ELF requirements. This is **not** proof of perfect future package compatibility; rerun checks after upgrades.

## Backup and recovery

Before deployment, local rollback archives were retained for the original October dnsdist 2.1.3, LMDB 0.9.35, Quiche 0.24.5_7, and libnghttp2 1.70.0. A separate copy of the active dnsdist configuration and a copy of the `pkg` database were created. SQLite `PRAGMA integrity_check` returned `ok` on that backup.

**Do not blindly restore the copied pkg SQLite database.** The supported recovery concept is to stop dnsdist, use the verified rollback archives as an internally consistent set, restore configuration only if needed, recheck dependencies and listeners, and start dnsdist. Any rollback involving a switch from `lmdb0` to `lmdb` must remove the conflicting provider in the package transaction; test the exact rollback plan first. No full live rollback test was recorded for this November deployment.

## Post-installation checks (2026-11-18)

- `dnsdist --version`: **2.1.3 (Lua 5.4.8)**; DoT, DoH, DoQ, DoH3, DNSCrypt, YAML, fstrm and LMDB among enabled features.
- `dnsdist --check-config -C ...`: **Configuration OK**.
- `pkg check -d -n`: completed at **100%** without reported errors.
- Services: dnsdist **running**; Technitium **running**, not restarted during the package transaction; Unbound was not running.
- DNS A lookups for a public test domain via dnsdist loopback **UDP and TCP** returned **NOERROR**.
- DNS A lookups via configured **LAN and VLAN** listeners returned **NOERROR**.
- Direct query to the Technitium non-PROXY listener returned **NOERROR**.
- Listening sockets showed dnsdist on the configured loopback, LAN, VLAN and other interfaces; Technitium on dedicated local DNS ports.
- The dnsdist backend configuration includes `useProxyProtocol = true`, but no separate evidence was captured that the client's original IP is recorded correctly by Technitium. The DNS resolution checks alone do not prove PROXY Protocol propagation.
- The command `dnsdist -c -e 'showServers()'` returned **“The currently configured console key is not valid”**. This concerns the management console, **not** the successful DNS resolution. No console key was changed during migration.
- No live verification of every encrypted transport, sustained load, or Internet-facing resolver exposure was recorded. Audit firewall rules separately where dnsdist listens on WAN interfaces.

## Maintenance and next steps

- Retain local, verified rollback archives until future upgrades are proven.
- Recheck OPNsense packages and SONAMEs before reusing the same Poudriere binaries on another version of FreeBSD or OPNsense.
- Consider formalizing the jail, ports commit, options, package manifest checks and install dry-run as scripts in this repository, but do not claim full byte-for-byte reproducibility yet.
- Manage the optional dnsdist console authentication separately.
- If publishing a November artifact, use a **new release/tag and manifest**, not the October `v2.1.3-opnsense-freebsd15-amd64` asset or its SHA256.
