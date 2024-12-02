sbatch # To run a *.sh script
squeue # To see the lists of current jobs running (or waiting) in the cluster
smap -i 1 # To see live the status of each job running (or waiting) in the cluster.
scancel # Cancels a job.


to check sizes of all directories
du -h --max-depth=1

```
854 second_job.slurm #to run a scrit after another job is finished

sbatch --dependency=afterok:[id first job] [name of second job].sh
sbatch --dependency=afterok:113424 /home/ubuntu/emiliano/2nd_chp/mapped_reads/RSBK01/bwa_RSBK01.sh
```


```
#SBATCH --error=%x.%j.%N.err

sudo scontrol update nodename=med128compute001 state=down reason=cg
sudo scontrol update nodename=med128compute001 state=resume
```

sinfo --Node --long # To get information of each node in the cluster. Example below.
#STATE can be allocated (in current use), mixed or idle (no used).

```
Mon Apr 25 10:58:25 2022
NODELIST          NODES PARTITION       STATE CPUS    S:C:T MEMORY TMP_DISK WEIGHT AVAIL_FE REASON              
compute001            1     main*       mixed   32   32:1:1 249553        0      1   (null) none                
computeGPU001         1     main*   allocated   16   16:1:1  62526        0      1   (null) none                
computeGPU002         1     main*        idle   16   16:1:1  62526        0      1   (null) none                
med32compute001       1     main*        idle    8    8:1:1  31027        0      1   (null) none                
med32compute002       1     main*        idle    8    8:1:1  31027        0      1   (null) none                
med32compute003       1     main*        idle    8    8:1:1  31027        0      1   (null) none                
med32compute004       1     main*        idle    8    8:1:1  31027        0      1   (null) none                
med64compute001       1     main*        idle   16   16:1:1  62526        0      1   (null) none                
med64compute002       1     main*        idle   16   16:1:1  62526        0      1   (null) none                
med64compute003       1     main*        idle   16   16:1:1  62526        0      1   (null) none                
med64compute004       1     main*        idle   16   16:1:1  62526        0      1   (null) none                
med128compute001      1     main*        idle   32   32:1:1 125523        0      1   (null) none                
med128compute002      1     main*        idle   32   32:1:1 125523        0      1   (null) none                
smallcompute001       1     main*        idle    4    4:1:1  15278        0      1   (null) none                
smallcompute002       1     main*        idle    4    4:1:1  15278        0      1   (null) none
```

NEW configuration (Jaunary 2024)

Thu Feb  1 09:49:46 2024
NODELIST          NODES PARTITION       STATE CPUS    S:C:T MEMORY TMP_DISK WEIGHT AVAIL_FE REASON              
compute001            1     main*       mixed   32   32:1:1 249553        0      1   (null) none                
computeGPU001         1     main*    drained*   16   16:1:1  62526        0      1   (null) Maintain            
computeGPU002         1     main*       down*   16   16:1:1  62526        0      1   (null) Not responding      
med32compute001       1     main*        idle    8    8:1:1  31027        0      1   (null) none                
med32compute002       1     main*        idle    8    8:1:1  31027        0      1   (null) none                
med64compute001       1     main*        idle   16   16:1:1  62526        0      1   (null) none                
med64compute002       1     main*        idle   16   16:1:1  62526        0      1   (null) none                
med128compute001      1     main*        idle   32   32:1:1 125523        0      1   (null) none                
med128compute002      1     main*        idle   32   32:1:1 125523        0      1   (null) none

compute001
249553
32
7800
computeGPU001
62526
16
3800
computeGPU002
62526
16
3800
med32compute001
31027
8
3800
med32compute002
31027
8
3800
med32compute003
31027
8
3800
med32compute004
31027
8
3800
med64compute001
62526
16
3800
med64compute002
62526
16
3800
med64compute003
62526
16
3800
med64compute004
62526
16
3800
med128compute001
125523
32
3800
med128compute002
125523
32
3800
smallcompute001
15278
4
3800
smallcompute002
15278
4
3800


```
#!/bin/bash
#SBATCH --job-name=parallel_job      # Job name
#SBATCH --mail-type=END,FAIL         # Mail events (NONE, BEGIN, END, FAIL, ALL)
#SBATCH --mail-user=email@ufl.edu    # Where to send mail	
#SBATCH --nodes=1                    # Run all processes on a single node	
#SBATCH --ntasks=1                   # Run a single task		
#SBATCH --cpus-per-task=4            # Number of CPU cores per task
#SBATCH --mem=1gb                    # Job memory request
#SBATCH --time=00:05:00              # Time limit hrs:min:sec
#SBATCH --output=parallel_%j.log     # Standard output and error log

#SBATCH --mail-user=emiliano.mora@systbot.uzh.ch
```

Exclude some nodes:
```
#SBATCH --exclude=computeGPU001,computeGPU002,compute001
```

df -h

find . -type f -name "datadict.txt" > datadict_files
