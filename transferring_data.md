# Transferring Data: rsync vs scp

You can use `rsync` or `scp` to transfer files between locations. [Here](https://stackoverflow.com/questions/20244585/how-does-scp-differ-from-rsync) are some reasons why `rsync` is generally preferred over `scp`. Basically, `rsync` is more efficient and can resume transfers if interrupted.

## rsync

`rsync` transfers files from one directory to another within the same shell location or between different (remote) locations.

### Basic syntax

```bash
rsync -varzP source_dir/ target_dir/
```

### Options

- `-v` (verbose): Increases verbosity/information output of the operation
- `-a` (archive): Preserves permissions, timestamps, symbolic links, and other important file data
- `-r` (recursive): Copies directories recursively (redundant if using `-a`)
- `-z` (compress): Compresses data during transfer to reduce bandwidth usage; useful for slower connections
- `-P` (progress and partial): Shows progress during transfer and keeps partially transferred files; useful for large files and transfers that might be interrupted (equivalent to `--partial --progress`)

### Remote to remote rsync

To rsync between two remote shell locations:

```bash
rsync -avz -e "ssh -i .ssh/NAME_OF_KEY" source_dir/ ubuntu@IP_ADDRESS:target_dir/
```

## scp

`scp` is a simpler, more straightforward tool for copying files between hosts over SSH.

### Local to remote

```bash
scp /source_directory/file.txt ubuntu@IP_ADDRESS:target_dir/
```

### Remote to local

```bash
scp ubuntu@IP_ADDRESS:source_dir/file.txt /target_directory/
```

### Remote to remote

```bash
scp username@remote_1:/file/to/send username@remote_2:/where/to/put
```

### Example with network share

```bash
scp /run/user/$UID/gvfs/smb-share\:server\=idnas37.d.uzh.ch,share\=g_systbot_metabolomics$/WGS/F24A910000236-01_ERYtrjgR_20250822035936/1/1.png ubuntu@172.23.85.230:/home/ubuntu/emiliano
```
