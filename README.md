# dir-backup

`dir-backup` packs files and directories into a `tar + gzip` archive, locally or
streamed to a remote host over SSH, without writing the archive to disk on the
network path.

It provides two subcommands:

- `archive` — pack one or more local files/directories into a `.tar.gz`, written
  locally or streamed to a remote host (optionally extracted there with `--extract`).
- `unarchive` — extract a local or remote `.tar.gz`, optionally remapping path prefixes.
  Remote archives are streamed over SSH and never downloaded to local disk.

## Install

### With mise

```bash
mise use --global github:parisni/dir-backup@v1.0.0
```

### From release asset

1. Download the right archive for your platform (`linux_amd64` or `linux_arm64`).
2. Extract it:

```bash
tar -xzf dir-backup_vX.X.X_linux_amd64.tar.gz
chmod +x dir-backup
```

3. Move it into your `PATH` (example):

```bash
sudo mv dir-backup /usr/local/bin/dir-backup
```

## Usage

```bash
dir-backup archive [--extract [--remap OLD_PREFIX:NEW_PREFIX] [--dest /remote_dir]] [--owner OWNER] [--group GROUP] <output> <abs_path1> [<abs_path2> ...]
dir-backup unarchive [--remap OLD_PREFIX:NEW_PREFIX] <archive.tar.gz | user@host:/abs/path/file.tar.gz> [<dest_dir>]
```

### `archive`

- Packs one or more files or directories into a `.tar.gz`.
- Source paths must be absolute.
- `<output>` is local when absolute (`/path.tar.gz`) and remote when a remote-spec (`user@host:/path.tar.gz`); a leading `/` always means local.
- A remote `<output>` streams the data and writes the raw `.tar.gz` remotely (no local copy is kept).
- `--extract` (remote only) extracts the stream on the remote host instead of writing the archive. Paths are stored relative to `/`, restoring absolute locations; `--dest /dir` extracts relative to a directory and `--remap OLD:NEW` rewrites a prefix.
- `--owner`/`--group` force ownership when extracting remotely (effective only when the SSH user is `root`).
- Paths are stored relative to `/` so `unarchive` can restore them to their original absolute locations.

### `unarchive`

- Source can be a local archive (`/abs/path/file.tar.gz`) or a remote one (`user@host:/abs/path/file.tar.gz`).
- Remote sources are streamed over SSH (`ssh host 'cat …' | tar -xzf -`) and are never written to local disk.
- Without `<dest_dir>`, files are restored to their original absolute paths (extraction relative to `/`).
- With `<dest_dir>`, files are extracted relative to that directory.
- `--remap OLD_PREFIX:NEW_PREFIX` rewrites a leading path prefix at extraction time via `tar --transform`. Both prefixes are interpreted relative to the archive layout (leading `/` is stripped automatically).

## Examples

Stream the raw `.tar.gz` to a remote host:

```bash
dir-backup archive user@host:/tmp/backup.tar.gz /tmp/gimp
```

Stream and extract remotely, forcing ownership/group (effective only when SSH user is `root`):

```bash
dir-backup archive --extract --owner backup --group backup user@host:/ignored.tar.gz /tmp/gimp
```

Create a local archive of multiple absolute paths:

```bash
dir-backup archive /tmp/backup.tar.gz /etc/nginx /home/me/notes
```

Extract a local archive back to its original absolute paths:

```bash
dir-backup unarchive /tmp/backup.tar.gz
```

Extract a local archive into a different directory:

```bash
dir-backup unarchive /tmp/backup.tar.gz /tmp/restore
```

Remap a path prefix during extraction (e.g. moving a home directory):

```bash
dir-backup unarchive --remap /home/nparis:/home/parisni /tmp/backup.tar.gz
```

Extract a remote archive locally without downloading it first (streamed over SSH):

```bash
dir-backup unarchive user@host:/srv/backups/site.tar.gz /tmp/restore
```

Same, with a path remap applied on the fly:

```bash
dir-backup unarchive --remap /home/nparis:/home/parisni \
  user@host:/srv/backups/site.tar.gz /tmp/restore
```

## Notes on ownership

- `archive --extract`'s `--owner` / `--group` only take effect when the SSH user on the remote is `root`. Otherwise extracted files are owned by the SSH connection user.
- `unarchive` does not currently expose `--owner` / `--group`; extracted files are owned by the local user running the command (or preserved from the archive when run as `root`, per `tar`'s defaults).
