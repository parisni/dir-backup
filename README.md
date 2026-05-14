# dir-backup

`dir-backup` streams directories and files between hosts using `tar + gzip`,
without ever writing the archive to disk on the network path.

It provides three subcommands:

- `scp` — stream a local directory to a remote host and extract it there in one pass.
- `archive` — pack one or more local files/directories into a local `.tar.gz`.
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
dir-backup scp [--owner OWNER] [--group GROUP] <local_abs_path> <user@host:/remote_abs_path>
dir-backup archive <output.tar.gz> <abs_path_or_file1> [<abs_path_or_file2> ...]
dir-backup unarchive [--remap OLD_PREFIX:NEW_PREFIX] <archive.tar.gz | user@host:/abs/path/file.tar.gz> [<dest_dir>]
```

### `scp`

- Directory-only; `local_abs_path` must be an absolute directory path.
- Data is streamed: local `tar | gzip` → SSH → remote `gzip -d | tar -x`.
- Remote destination is created if missing.
- `--owner` and `--group` apply only when the SSH user is `root`. Otherwise files are owned by the SSH connection user.

### `archive`

- Packs one or more files or directories into a local `.tar.gz`.
- Source paths and the output path must be absolute.
- Paths are stored relative to `/` so `unarchive` can restore them to their original absolute locations.

### `unarchive`

- Source can be a local archive (`/abs/path/file.tar.gz`) or a remote one (`user@host:/abs/path/file.tar.gz`).
- Remote sources are streamed over SSH (`ssh host 'cat …' | tar -xzf -`) and are never written to local disk.
- Without `<dest_dir>`, files are restored to their original absolute paths (extraction relative to `/`).
- With `<dest_dir>`, files are extracted relative to that directory.
- `--remap OLD_PREFIX:NEW_PREFIX` rewrites a leading path prefix at extraction time via `tar --transform`. Both prefixes are interpreted relative to the archive layout (leading `/` is stripped automatically).

## Examples

Copy local directory contents into a remote target directory:

```bash
dir-backup scp /tmp/gimp/ user@host:/tmp/gimp/
```

Try forcing ownership/group during extraction (effective only when SSH user is `root`):

```bash
dir-backup scp --owner backup --group backup /tmp/gimp/ user@host:/tmp/gimp/
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

- `scp`'s `--owner` / `--group` only take effect when the SSH user on the remote is `root`. Otherwise extracted files are owned by the SSH connection user.
- `unarchive` does not currently expose `--owner` / `--group`; extracted files are owned by the local user running the command (or preserved from the archive when run as `root`, per `tar`'s defaults).
