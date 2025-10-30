# One can use rsync or scp
[Here](https://stackoverflow.com/questions/20244585/how-does-scp-differ-from-rsync), I found some reasons of why rsync is better than scp. Basically, rsync is more efficient and can resume the transfer if the conecction is broken, whereas scp cannot.

# rsync
rsync is to transfer files from one directory to the other within the same shell location or from two different (remote) locations.
```
rsync -varzP source_dir/ target_dir/

-v (verbose): Increases verbosity or information output of the operation.
-a (archive): Preserves permissions, timestamps, symbolic links, and other important file data.
-r (recursive): Copies directories recursively. Redundant if you use -a above.
-z (compress): Compresses the data as it is sent to the destination machine, which reduces the amount of data being transmitted. Useful for slower connections.
-P (progress and partial): Shows progress during transfer and keeps partially transferred files which is useful for large files and long transfers that might be interrupted. It's equivalent to --partial --progress.
```
To to rsync from two remote shell locations use the following command:
```
rsync -avz -e "ssh -i .ssh/NAME_OF_KEY" source_dir/ ubuntu@IP_ADDRESS:target_dir/
```
# scp
```
scp /run/user/$UID/gvfs/smb-share\:server\=idnas37.d.uzh.ch,share\=g_systbot_metabolomics$/WGS/F24A910000236-01_ERYtrjgR_20250822035936/1/1.png ubuntu@172.23.85.230:/home/ubuntu/emiliano
#From local computer to remote computer
scp /source_directory/file.txt ubuntu@IP_ADDRESS:target_dir/
#From remote computer to local
scp ubuntu@IP_ADDRESS:source_dir/file.txt /target_directory/
#From remote to remote
scp username@remote_1:/file/to/send username@remote_2:/where/to/put
```
