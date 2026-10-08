# Installation and rollback

These instructions are intended for experienced OPNsense / FreeBSD administrators.

The package is an **unofficial compatibility repack**. Keep console or other recovery access available when changing DNS infrastructure.

## 1. Verify the target

Check the operating system and architecture:

```sh
freebsd-version
uname -m
pkg -vv | grep -E '^(ABI|ALTABI)'
```

The package metadata targets:

```text
ABI:    FreeBSD:15:amd64
ALTABI: freebsd:15:x86:64
```

Do not force-install it on a different ABI without independently validating compatibility.

## 2. Verify the package before installation

Assuming the release asset has been downloaded as:

```text
dnsdist-2.1.2-opnsense-freebsd15-amd64.pkg
```

calculate its checksum:

```sh
sha256 dnsdist-2.1.2-opnsense-freebsd15-amd64.pkg
```

Expected SHA256:

```text
09955ca8409146b1f440365ec43c6434b4802a02d321ec9ec37f65172b6d20a2
```

Inspect package metadata before installation:

```sh
pkg info -F dnsdist-2.1.2-opnsense-freebsd15-amd64.pkg
pkg info -F dnsdist-2.1.2-opnsense-freebsd15-amd64.pkg -R
```

Confirm that the package identity, ABI, dependencies and local repack annotation match the documentation in this repository.

## 3. Record the current state

Before replacing an existing dnsdist installation:

```sh
dnsdist --version
pkg info dnsdist
service dnsdist status
```

Save the current package if it is installed:

```sh
mkdir -p /root/dnsdist-backup
pkg create -o /root/dnsdist-backup dnsdist
```

Back up dnsdist configuration separately:

```sh
stamp=$(date +%Y%m%d-%H%M%S)
tar -C /usr/local/etc -czf "/root/dnsdist-config-${stamp}.tar.gz" dnsdist
```

If your installation uses additional certificates, Lua files, include directories or generated configuration outside `/usr/local/etc/dnsdist`, back those up separately.

## 4. Check dependencies

The package manifest declares these package dependencies:

```text
gnutls
libedit
libnghttp2
libsodium
lmdb
lua54
quiche
re2
tinycdb
```

The specific versions recorded in the release manifest are documented in [REPACK.md](REPACK.md).

Before installation, verify that the configured repositories can provide the required dependencies or that compatible versions are already installed.

Useful check:

```sh
pkg info | egrep '^(gnutls|libedit|libnghttp2|libsodium|lmdb|lua54|quiche|re2|tinycdb)-'
```

The binary also requires OpenSSL 3.5 shared libraries:

```text
libssl.so.35
libcrypto.so.35
```

Verify them on the target system before replacing a working DNS service.

## 5. Install

Stop dnsdist during the package replacement:

```sh
service dnsdist stop
```

Install the local package:

```sh
pkg add -f ./dnsdist-2.1.2-opnsense-freebsd15-amd64.pkg
```

Then start the service:

```sh
service dnsdist start
```

## 6. Validate

Check package and binary versions:

```sh
pkg info dnsdist
dnsdist --version
```

Check service state:

```sh
service dnsdist status
```

Inspect listeners:

```sh
sockstat -4 -6 -l | grep dnsdist
```

Perform a DNS query through the listener used by your deployment, for example:

```sh
drill @127.0.0.1 example.com A
```

Use the actual listener address and port from your configuration when it differs from `127.0.0.1:53`.

Also validate any encrypted DNS listeners or frontend/backend paths that your own dnsdist configuration provides. Their availability depends on your configuration, certificates and runtime environment, not merely on the package being installed.

## 7. Configuration behavior to be aware of

This repack disables automatic lifecycle handling for:

```text
dnsdist.yml.sample
```

The YAML sample remains packaged, but it is not automatically copied into a live YAML configuration and is not automatically removed by the repack's package scripts.

The traditional `dnsdist.conf.sample` lifecycle handling remains in the package.

Before installation or removal, keep independent backups of live configuration.

## 8. Rollback

If validation fails:

```sh
service dnsdist stop
```

Locate the backup package created earlier:

```sh
ls -lh /root/dnsdist-backup/
```

Restore the previous package with `pkg add -f`, using the exact backup filename:

```sh
pkg add -f /root/dnsdist-backup/<previous-dnsdist-package>.pkg
```

Restore configuration from the backup archive only if required.

Then:

```sh
service dnsdist start
service dnsdist status
dnsdist --version
```

Finally, test DNS resolution again through the actual production listener.

## OPNsense note

OPNsense upgrades can replace packages or alter dependency versions. Treat this local package as an explicitly managed local modification and revalidate it after major OPNsense / FreeBSD upgrades.
