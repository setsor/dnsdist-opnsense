# dnsdist 2.1.2 — OPNsense / FreeBSD 15 repack notes

This document describes the local package adaptation represented by the release in this repository.

## Package identity

```text
Name:         dnsdist
Version:      2.1.2
Origin:       dns/dnsdist
ABI:          FreeBSD:15:amd64
Architecture: freebsd:15:x86:64
Prefix:       /usr/local
```

The package SHA256 is:

```text
09955ca8409146b1f440365ec43c6434b4802a02d321ec9ec37f65172b6d20a2
```

## Local repack annotation

The package manifest records the following annotation:

```text
Original 2.1.2 binary; legacy file metadata; LMDB 0.9.35 dependency mapped to OPNsense lmdb; YAML sample auto-copy/removal disabled
```

That annotation defines the intended scope of the repack.

## Changes

### 1. dnsdist binary retained

The repack records the dnsdist 2.1.2 executable as the original binary.

No source-code modification to dnsdist itself is claimed by this repository.

### 2. LMDB dependency mapping

The package dependency metadata uses:

```text
lmdb
origin: databases/lmdb
version: 0.9.35,1
```

This matches the LMDB package used by the tested OPNsense environment while retaining the runtime requirement on:

```text
liblmdb.so.0
```

### 3. YAML sample lifecycle disabled

The automatic package scripts associated with `dnsdist.yml.sample` were removed from the repacked manifest.

The purpose is to prevent package installation or deinstallation from automatically creating or removing a YAML configuration derived from the sample file.

The package still contains:

```text
/usr/local/etc/dnsdist/dnsdist.yml.sample
```

but the repack does not automatically copy it to a live YAML configuration or remove such a configuration.

### 4. Traditional dnsdist.conf behavior retained

The package lifecycle script for:

```text
/usr/local/etc/dnsdist/dnsdist.conf.sample
```

remains present.

On installation, if the corresponding target configuration does not exist, the package script can copy the sample to the target.

On deinstallation, the package script can remove the target only when it is unchanged from the sample; otherwise it leaves the file in place and may print a notice.

### 5. Legacy file metadata retained

The repack annotation explicitly records that legacy file metadata from the source package was preserved.

No broader normalization of packaged files is claimed.

## What was not changed

This repack does **not** claim:

- a custom dnsdist source patch;
- a custom dnsdist version number;
- a rebuilt dnsdist executable;
- a change to the upstream product identity;
- official support from PowerDNS;
- official support from OPNsense.

## Package options

The manifest records:

```text
CDB      on
DNSTAP   off
GNUTLS   on
IPCIPHER on
LMDB     on
LUA      on
LUAJIT   off
OPENSSL  on
SNMP     off
```

## Runtime libraries recorded by the package

```text
libc++.so.1
libc.so.7
libcdb.so.1
libcrypto.so.35
libcxxrt.so.1
libedit.so.0
libgcc_s.so.1
libgnutls.so.30
liblmdb.so.0
liblua-5.4.so
libm.so.5
libnghttp2.so.14
libquiche.so.0
libre2.so.11
libsodium.so.26
libssl.so.35
libthr.so.3
```

## Declared package dependencies

The package manifest records these dependency versions:

| Package | Origin | Version |
| --- | --- | --- |
| gnutls | security/gnutls | 3.8.13 |
| libedit | devel/libedit | 3.1.20260512,1 |
| libnghttp2 | www/libnghttp2 | 1.70.0 |
| libsodium | security/libsodium | 1.0.22 |
| lmdb | databases/lmdb | 0.9.35,1 |
| lua54 | lang/lua54 | 5.4.8 |
| quiche | net/quiche | 0.24.5_9 |
| re2 | devel/re2 | 20251105 |
| tinycdb | databases/tinycdb | 0.81 |

These values document the package metadata at the time of the repack. They should not be interpreted as a promise that future OPNsense repositories will retain exactly these versions.
