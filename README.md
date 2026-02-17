# dir-backup

`dir-backup` streams a local directory to a remote host over SSH using `tar + gzip` in one pass.

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
```

Notes:
- `scp` is directory-only.
- `local_abs_path` must be an absolute directory path.
- Data is streamed: local `tar | gzip` -> SSH -> remote `gzip -d | tar -x`.
- Remote destination is created if missing.

## Examples

Copy local directory contents into remote target directory:

```bash
dir-backup scp /tmp/gimp/ natus:/tmp/gimp/
```

Try forcing ownership/group during extraction:

```bash
dir-backup scp --owner backup --group backup /tmp/gimp/ natus:/tmp/gimp/
```

Important:
- Without root privileges on remote, ownership changes may not apply.

