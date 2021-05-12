# ZeroNS: a name service centered around the ZeroTier Central API

ZeroNS provides names that are a part of [ZeroTier Central's](https://my.zerotier.com) configured _networks_; once provided a network it:

- Listens on the local interface joined to that network -- you will want to start one ZeroNS per ZeroTier network.
- Provides general DNS by forwarding all queries to `/etc/resolv.conf` resolvers that do not match the TLD, similar to `dnsmasq`.
- Tells Central to point all clients that have the "Manage DNS" settings turned **on** to resolve to it.
- Finally, sets a provided TLD (`.domain` is the default), as well as configuring `A` (IPv4) and `AAAA` (IPv6) records for:
  - Member IDs: `zt-<memberid>.<tld>` will resolve to the IPv4/v6 addresses for them.
  - Names: _if_ the names are compatible with DNS names, they will be converted as such: to `<name>.<tld>`.
    - Please note that **collisions are possible** and that it's _up to the admin to prevent them_.

## Installation

Please obtain a working [rust environment](https://rustup.rs/) first.

### Get a release from Cargo

```
cargo install zeronsd
```

### From Git

```
cargo install --git https://github.com/zerotier/zeronsd --branch main
```

### Docker

There is a `Dockerfile` present in the repository you can use to build images in lieu of one of our official images (*N.B.:* these don't exist yet, for now, please build your own).

There are two build arguments which control behavior:

- `VERSION`: this is the branch or tag to fetch.
- `IS_TAG`: if non-zero, tells cargo to fetch tags instead of branches.

Example:

```bash
docker build . # builds latest master
docker build --build-arg VERSION=somebranch # builds branch `somebranch`
docker build --build-arg IS_TAG=1 VERSION=v0.1.0 # builds version 0.1.0 from tag v0.1.0
```

Once built, the image automatically runs `zeronsd` for you. The default subcommand is `help`.

## Usage

Setting `ZEROTIER_CENTRAL_TOKEN` in the environment is required. You must be able to administer the network to use `zeronsd` with it. Also, running as `root` is required as _many client resolvers do not work over anything but port 53_. Your `zeronsd` instance will listen on both `udp` and `tcp`, port `53`.

### Bare commandline

**Tip**: running `sudo`? Pass the `-E` flag to import your current shell's environment, making it easier to add the `ZEROTIER_CENTRAL_TOKEN`.

```
zeronsd start <network id>
```

### Docker

Running in docker is a little more complicated. You must be able to have a network interface you can import (joined a network) and must be able to reach `localhost:9999` on the host. At this time, for brevity's sake we are recommending running with `--net=host` until we have more time to investigate a potentially more secure solution.

```
docker run --net host zeronsd start <network id>
```

### Other notes

You must have already joined a network and obviously, `zerotier-one` should be running!

It should print some diagnostics after it has talked to your `zerotier-one` instance to figure out what IP to listen on. After that it should communicate with the central API and set everything else up automatically.

### Flags

- `-d <tld>` will set a TLD for your records; the default is `domain`.
- `-f <hosts file>` will parse a file in `/etc/hosts` format and append it to your records.
- `-s <secret file>` path to `authtoken.secret` which is needed to talk to ZeroTier on localhost. You can provide this file with this argument, but it is auto-detected on multiple platforms including Linux, OS X and Windows.
- `-t <central token file>` path to file containing your [ZeroTier Central token](https://my.zerotier.com/account).

### TTLs

Records currently have a TTL of 60s, and Central's records are refreshed every 30s through the API. I felt this was a safer bet than letting timeouts happen.

### Per-Interface DNS resolution

OS X and Windows users get this functionality by default, so there is no need for it.

Linux users are strongly encouraged to use `systemd-networkd` along with `systemd-resolved` to get per-interface resolvers that you can isolate to the domain you want to use. If you'd like to try something that can assist with getting you going quickly, check out the [erikh/zerotier-systemd-manager repository](https://github.com/erikh/zerotier-systemd-manager).

BSD systems still need a bit of work; work that we could really use your help with if you know the lay of the land on your BSD of choice. Set up an issue if this interests you.

## Acknowledgements

ZeroNS demands a lot out of the [trust-dns](https://github.com/bluejekyll/trust-dns) toolkit and I personally am grateful such a library suite exists. It made my job very easy.

## Author

Erik Hollensbe <github@hollensbe.org>
