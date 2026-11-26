# openwrt-n2n-v3

OpenWrt packaging for the [ntop/n2n](https://github.com/ntop/n2n) v3 layer-two peer-to-peer VPN.

This repository does not contain the n2n source code itself. It provides a single OpenWrt package definition that builds and installs the `edge` and `supernode` daemons, along with default configuration files and `procd` init scripts.

## Features

- Separate OpenWrt packages for `n2n-edge` and `n2n-supernode`
- Tracks the upstream `ntop/n2n` `dev` branch; declared package version is `3.1.1`
- OpenSSL enabled (`--with-openssl`)
- Optional `zstd` and `libcap` detection disabled by default for portability across OpenWrt 19.07–25.12 build environments
- `procd` service management with configuration-file monitoring and automatic respawn
- Configuration files installed as `conffiles`, so they are preserved across upgrades/sysupgrade

## Packages

| Package | Binary | Config | Init script | Dependencies |
| --- | --- | --- | --- | --- |
| `n2n-edge` | `/usr/sbin/edge` | `/etc/n2n/edge.conf` | `/etc/init.d/n2n-edge` | `kmod-tun`, `libpthread`, `libopenssl` |
| `n2n-supernode` | `/usr/sbin/supernode` | `/etc/n2n/supernode.conf` | `/etc/init.d/n2n-supernode` | `libpthread`, `libopenssl` |

## Requirements

- An OpenWrt buildroot
- Target support for TUN interfaces when using `n2n-edge` (`kmod-tun`)
- The `libpthread` and `libopenssl` packages available from the configured feeds

## Adding the package to OpenWrt

### Option A: copy or symlink into the buildroot

```sh
ln -s /absolute/path/to/openwrt-n2n-v3/package/n2n-v3 <openwrt-root>/package/n2n-v3
cd <openwrt-root>
./scripts/feeds update -a
./scripts/feeds install -a
```

### Option B: add it as a feed

```sh
echo 'src-link n2n-v3 /absolute/path/to/openwrt-n2n-v3/package' >> <openwrt-root>/feeds.conf.default
cd <openwrt-root>
./scripts/feeds update n2n-v3
./scripts/feeds install -a -p n2n-v3
```

Then select the packages:

```sh
make menuconfig
```

Navigate to:

```text
Network  --->
  VPN  --->
    <*> n2n-edge
    <*> n2n-supernode
```

Build the firmware or the package target:

```sh
make -j$(nproc)
# or, when placed directly under package/:
make -j$(nproc) package/n2n-v3/compile
```

## Configuration

Configuration files use one option per line with an `=` between the option and its value. Run `edge -h` or `supernode -h` for the full option list.

The shipped files are examples only. Replace all placeholder community names, keys, addresses, and supernode hosts before use.

### Supernode

Edit `/etc/n2n/supernode.conf`:

```text
-p=7777
```

`-p` is the UDP listening port. Optionally, `-c=community.list` can restrict allowed communities; the `community.list` file is not installed by this package and must be supplied separately.

Enable and start the service:

```sh
/etc/init.d/n2n-supernode enable
/etc/init.d/n2n-supernode start
```

Make sure the chosen UDP port is allowed through the device firewall.

### Edge

Edit `/etc/n2n/edge.conf`:

```text
-d=n2n0
-c=mynetwork
-k=mypassword
-a=1.2.3.4
-p=50001
-l=7.8.9.0:7777
```

| Option | Meaning |
| --- | --- |
| `-d` | TUN interface name, for example `n2n0` |
| `-c` | Community name; must match the other edges and the supernode policy |
| `-k` | ASCII encryption key; `N2N_KEY=<key>` can also be used |
| `-m` | Optional fixed MAC address; random if omitted |
| `-a` | Interface address; use `-r` with `-a=dhcp:0.0.0.0` for DHCP |
| `-p` | Fixed local UDP port |
| `-l` | Supernode `host:port` |

Each edge should use a unique address, MAC address, and local UDP port. The community, encryption key, and supernode address must be consistent across the network.

Enable and start the service:

```sh
/etc/init.d/n2n-edge enable
/etc/init.d/n2n-edge start
```

If a fixed local UDP port is used and the edge is behind a firewall or NAT, allow or forward that port as needed.

## Repository layout

```text
package/n2n-v3/
├── Makefile
└── files/
    ├── edge.conf
    ├── supernode.conf
    ├── n2n-edge.init
    └── n2n-supernode.init
```

## Notes

- The init scripts skip startup if the corresponding config file is missing.
- Services run in the foreground with `-f` under `procd`, with respawn enabled.
- Config files are registered as `conffiles`, so custom settings survive package upgrades.
- Logs can be inspected with `logread -e edge` and `logread -e supernode`.
- `Makefile` currently tracks upstream `dev` and skips source mirror hash verification (`PKG_MIRROR_HASH:=skip`). Pin `PKG_SOURCE_VERSION` and set a real mirror hash for reproducible builds.
- To enable `zstd`, add `+libzstd` to the relevant `DEPENDS` and remove `ac_cv_lib_zstd_ZSTD_compress=no` from `CONFIGURE_VARS`.

## License

GPL-3.0-only, matching the upstream n2n project and the license declared in `package/n2n-v3/Makefile`.
