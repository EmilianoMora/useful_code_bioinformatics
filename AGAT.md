## Uninstall AGAT

```sh
conda uninstall agat
```

Install AGAT again to get the newest version
```
conda activate emiliano3.6
conda install -c bioconda agat
git clone https://github.com/NBISweden/AGAT.git 
cd AGAT                                         
perl Makefile.PL                                
make                                            
make test                                       
make install
```

## Run AGAT
```sh
conda activate emiliano3.6
#To a GTX to GFF3.
/home/ubuntu/etienne/progs/miniconda3/envs/emiliano3.6/bin/agat_convert_sp_gxf2gxf.pl -g pve_haplotypeT.final.gtf --gvi 2.5 --gvo 3 -o pve.gff
conda deactivate
```
