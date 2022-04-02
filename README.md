# 🪐 gocryptfs-rsync

> Helper to rsync over [gocryptfs] (encrypted rsync).

[gocryptfs]: https://github.com/rfjakob/gocryptfs

## Dependencies

rsync, [gocryptfs] and `python3` are required.

```sh
apt install rsync gocryptfs python3
pacman -S rsync gocryptfs python
brew install rsync
```

For gocryptfs on macOS, it's a bit more complicated now it's not part of
Homebrew anymore, [see instructions below](#gocryptfs-on-macos).

## Installation

The easiest is to just clone this repo and link the commands to a
directory that's in your `PATH`:

```sh
git clone https://github.com/valeriangalliat/gocryptfs-rsync
cd gocryptfs-rsync
ln -s "$PWD/gocryptfs-rsync" "$PWD/gocryptfs-rsync-pretty" ~/bin
```

## Usage

```
gocryptfs-rsync <src> <dest> [gocryptfs_options] -- [rsync_options]
```

To rsync `/path/to/foo` to `some-ssh-host:foo`. Both commands will
prompt for password.

```sh
gocryptfs --reverse --init /path/to/foo # Only the first time
gocryptfs-rsync /path/to/foo some-ssh-host:foo -- -ai --delete --info=progress2
```

Refer to [`rsync(1)`](https://linux.die.net/man/1/rsync) for rsync
options (here, `-ai --delete --info=progress2`).

To do the same thing, specifying a custom password program and an
exclude file:

```sh
gocryptfs-rsync /path/to/foo some-ssh-host:foo --extpass your-program --exclude-from /path/to/foo/.rsyncignore -- -ai --delete --info=progress2
```

The great thing with gocryptfs is that it's exclude list syntax is the
exact same as rsync, making it trivial to migrate existing exclusions.

Alternative rsync options could be using `-v` (`--verbose`) instead of
`-i` (`--itemize-changes`), or not adding verbosity altogether.

In some cases other options like `--no-specials` and `--no-devices`
might come in handy.

## gocryptfs on macOS

First you need to install [macFUSE](https://osxfuse.github.io/):

```sh
brew install --cask macfuse
```

Then you (currently) need to build my [fork][gocryptfs-fork] of
[gocryptfs] against my other [fork][go-fuse-fork] of [go-fuse].

[go-fuse]: https://github.com/hanwen/go-fuse
[gocryptfs-fork]: https://github.com/valeriangalliat/gocryptfs/tree/fix-reverse-conf-darwin
[go-fuse-fork]: https://github.com/valeriangalliat/go-fuse/tree/poll-hack-read-only-darwin

```sh
git clone https://github.com/valeriangalliat/go-fuse
git -C go-fuse checkout poll-hack-read-only-darwin

git clone https://github.com/valeriangalliat/gocryptfs
cd gocryptfs

git checkout fix-reverse-conf-darwin
go mod edit -replace github.com/hanwen/go-fuse/v2=../go-fuse

./build-without-openssl.bash
```

Then make sure that `gocryptfs` and `gocryptfs-xray/gocryptfs-xray` are
in your `PATH`, e.g. linking them in a local `bin`:

```sh
ln -s "$PWD/gocryptfs" "$PWD/gocryptfs-xray/gocryptfs-xray" ~/bin
```

**Note:** if those forks are not online anymore, it probably means that
the [relevant](https://github.com/hanwen/go-fuse/pull/420)
[PRs](https://github.com/rfjakob/gocryptfs/pull/648) were merged! Build
from those project's main branch until it makes it to
[`gromgit`'s homebrew-fuse](https://github.com/gromgit/homebrew-fuse)
[formula](https://github.com/gromgit/homebrew-fuse/blob/main/Formula/gocryptfs-mac.rb)
(the best place to get FUSE filesystems on macOS since they've been
dropped from Homebrew).
