# dnsdist 2.1.3 — OPNsense / FreeBSD 15

## Status (2026-10-26)

**Built locally, installed, and operationally validated** on the production OPNsense firewall running FreeBSD 15.1-RELEASE-p4, amd64.

This is an independent build from the signed upstream dnsdist **2.1.3 source**, not a repack of the FreeBSD 2.1.2 binary. The previous 2.1.2 repack remains available for rollback.

**Release publishing status:** The package exists on the firewall; a GitHub Release asset has **not yet** been uploaded or published. Do not claim otherwise.

## Provenance

- Upstream source: https://downloads.powerdns.com/releases/dnsdist-2.1.3.tar.xz
- Detached signature: `dnsdist-2.1.3.tar.xz.asc`.
- GPG reported `Good signature` from Rémi Gacogne, key fingerprint `D6300CABCBF469BBE392E503A208ED4F8AF58446`; signer/key trust should be checked against an independent authoritative fingerprint.
- Archive integrity checked with `xz -t`.
- Build OS: OPNsense / FreeBSD 15.1 amd64.
- C/C++ compiler: FreeBSD Clang 19.1.7.
- Rust / Cargo for YAML: 1.99.1, target `x86_64-unknown-freebsd`.
- Build performed on the firewall, without using FreeBSD ports repositories and without `make install` over the running binary.

## Configure options and build

The following options were used:

```sh
cd /root/dnsdist-2.1.3-build/dnsdist-2.1.3
export PATH="/root/.cargo/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
export CFLAGS="-O2 -pipe -g0"
export CXXFLAGS="-O2 -pipe -g0"
export LIBCRYPTO_INCLUDES="-I/usr/include"
export LIBCRYPTO_LDFLAGS="-L/usr/lib"
export LIBCRYPTO_LIBS="-lcrypto"
export LIBSSL_CFLAGS="-I/usr/include"
export LIBSSL_LIBS="-L/usr/lib -lssl"

./configure \
  --prefix=/usr/local \
  --sysconfdir=/usr/local/etc/dnsdist \
  --with-libcrypto=/usr \
  --with-libssl \
  --enable-dnscrypt \
  --enable-dns-over-tls \
  --enable-dns-over-https \
  --enable-dns-over-quic \
  --enable-dns-over-http3 \
  --enable-yaml \
  --enable-ipcipher \
  --enable-ipcrypt2 \
  --with-gnutls \
  --with-nghttp2 \
  --with-quiche \
  --with-libsodium \
  --with-re2 \
  --with-cdb \
  --with-lmdb \
  --with-lua=lua-5.4
```

The first configuration attempt with `--with-lua=lua54` failed: on this FreeBSD system, the pkg-config module is named `lua-5.4`. The corrected configure command passed.

A clean build used `gmake clean` followed by `nice -n 10 gmake -j2`, with `CARGO_BUILD_JOBS=2`. No automatic updates of shared OPNsense dependencies were required.

### Important OpenSSL linking correction

Initial builds selected the OpenSSL ports package from `/usr/local/lib` (ABI `libssl.so.17` / `libcrypto.so.17`), although the flags mentioned `/usr/lib`. The reason was an earlier `-L/usr/local/lib` in the aggregate link command.

The executable was **relinked with absolute paths** to the base-system libraries, using the existing compiled objects:

```sh
touch dnsdist.o
gmake dnsdist \
  LIBSSL_LIBS="/usr/lib/libssl.so.35" \
  LIBCRYPTO_LDFLAGS="" \
  LIBCRYPTO_LIBS="/lib/libcrypto.so.35"
```

The resulting ELF `NEEDED` entries and `ldd` both confirmed `libssl.so.35` and `libcrypto.so.35`. Never modify system library symlinks to achieve this.

The compiled executable was copied to staging and stripped **only in staging**:

```sh
cp -p dnsdist /root/dnsdist-2.1.3-build/package-stage/usr/local/sbin/dnsdist
strip --strip-unneeded /root/dnsdist-2.1.3-build/package-stage/usr/local/sbin/dnsdist
```

Staged executable size: approximately 13 MiB; SHA256:

`cb52bda92b137d002da7ea7a0c97a72728927d84804bf6938ecbf27e601731cf`

## Package metadata and staging

The package was assembled using the existing `dnsdist-2.1.2` compatibility repack as the baseline for its FreeBSD rc service script and installation lifecycle scripts.

- `dnsdist.conf.sample`, `dnsdist.yml.sample`, `dnsdist.1` and `COPYING` were taken from upstream 2.1.3 source (manual page compressed to `dnsdist.1.gz`).
- The old FreeBSD `catalog.mk`, which named the 2.1.2 distfile, was omitted.
- License files from the previous package remain in staging; a full audit of bundled third-party license notices has **not** been independently completed.
- The package contains **9 files**.
- Manifest options include `DNSTAP: on`, because the binary was built with fstrm support.
- `shlibs_required` was refreshed from the actual 2.1.3 ELF `NEEDED` entries: 87 library names.
- There are 11 declared package dependencies, including `abseil 20250127.1_2`, `fstrm 0.6.1_1`, `quiche 0.24.5_7`, and OPNsense `lmdb 0.9.35,1`.
- The `_dnsdist` user/group creation script was retained; YAML sample auto-copy/removal stays disabled; Lua `dnsdist.conf.sample` lifecycle handling is retained.
- Package created locally with `pkg create -M manifest-2.1.3.json -r package-stage -o packages`.

Package path on build host:

`/root/dnsdist-2.1.3-build/packages/dnsdist-2.1.3.pkg`

Package SHA256:

`44ff2addaa44dfbe094628e7a3c16eb75991b88f136b34834c29e4902dae11c2`

Treat this digest as applying to **this exact local build**, not as an upstream PowerDNS checksum. Publish the binary and checksum together.

## Upgrade and rollback used

The installer `pkg install -nU ./packages/dnsdist-2.1.3.pkg` showed the local archive as a new package, though the original `dnsdist 2.1.2` was registered correctly. FreeBSD `pkg add` v2.8.4 lacks a dry-run `-n` option.

Before changing service state, the original 2.1.2 executable, original `dnsdist.conf`, and signed-off previous package were copied to an independent rollback directory and verified.

The successful upgrade sequence was:

```sh
service dnsdist stop
pkg add -f /root/dnsdist-2.1.3-build/packages/dnsdist-2.1.3.pkg
/usr/local/sbin/dnsdist --check-config -C /usr/local/etc/dnsdist/dnsdist.conf
service dnsdist start
```

`pkg add -f` reported that dnsdist was already installed, extracted the 2.1.3 package, and retained the existing `_dnsdist` user and group. The pre-upgrade configuration checksum was unchanged.

Rollback package:

`/root/dnsdist-2.1.2-packages/reusable/dnsdist-2.1.2.pkg`

Rollback package SHA256:

`09955ca8409146b1f440365ec43c6434b4802a02d321ec9ec37f65172b6d20a2`

In a failure, stop the service, restore the original package with `pkg add -f`, restore the backed-up configuration if altered, start dnsdist and test name resolution. Do not attempt to run two instances on the same listeners.

## Operational validation (2026-10-26)

- `pkg query` returned `dnsdist 2.1.3 dns/dnsdist`.
- `dnsdist --version` returned 2.1.3 and Lua 5.4.8, including DoQ, DoH3, DoT, DoH, DNSCrypt, YAML, ipcrypt2, fstrm, LMDB, GnuTLS and OpenSSL in enabled features.
- `service dnsdist status` reported a running daemon.
- UDP and TCP DNS listeners appeared on the configured local, LAN, and IPv6 interfaces.
- DNS A lookups through localhost and a LAN listener returned `NOERROR`.
- Active configuration checksum matched the pre-upgrade file exactly.
- `pkg check -d -n dnsdist` completed without errors.
- **Scope:** This verifies service startup, DNS over UDP on the tested listeners, package dependencies and configuration integrity. It does **not** independently establish successful live DoT/DoH/DoQ/DoH3/DNSCrypt sessions, load behavior, or full functional regression coverage.

## Maintenance notes

Do not commit live `dnsdist.conf`, secrets, addresses, backup archives or private network details. Package releases should include the binary package, its full SHA256, the manifest, provenance, and rollback notes.

This build record documents the commands actually used; it does not claim bit-for-bit reproducibility on a different machine or toolchain.
