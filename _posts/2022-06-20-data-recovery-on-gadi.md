## Data recovery on NCI-Gadi

Due to the New File Management Policy for NCI’s /scratch File System, data that are not accessed for 100 days will be quarantined. 

This blog presents some related codes to manage the files on gadi to address this issue.

---

### Gadi scratch file expiry commands

A [command sheet](https://nci.org.au/sites/default/files/documents/2022-04/GadiSystem-GadiScratchFileExpiryCommands-200422-1629-37.pdf) has been provided on nci website. In summary, ``` nci-file-expiry list-warnings``` shows all your files that will expire within 14 days. ```nci-file-expiry list-quarantined``` will provide a list of all your files which are recently expired and quarantined.
```bash
nci-file-expiry list-quarantined --project a00 > log_quarantined_a00
```

From the output record you can determine what to retrieved from the quarantined zone using ```nci-file-expiry recover UUID PATH```. 

#### Potential issues and scripts

- Too many quarantined files leading to system time out

If you have a large number of files (I don't know the bottom line but 1 million files can lead to this issue in my experience), the server can be timing out while trying to collect and process the information such as using ```nci-file-expiry recover```. In this case, you won't be able to get the list of your quarantined files and of course no way to recover them. You will need the help from NCI-Help-Desk. They would be helpful recovering your data or drop you a list of your quarantined files with UUID and PATH so that you can handle it yourself. We only talk about the latter case here.

- A large txt file needs to be split

If you have a large number of quarantined files, the txt file including the information will be large. If we are going to recover all files from quarantined, we'd better split large txt file into smaller ones so that later it would only take reasonably time to process the commands. The following command will do this, where 10000 is the number of lines in each smaller txt file.

```
split -l 10000 your-quarantined-list.txt
```

- Use arguments from a txt file line-by-line to execute a command

If you have a txt file documenting the following information where each line is for a quarantined file, such as,

```
someArbitraryCharactersForUUID1 theRelatedPATH1\n
someArbitraryCharactersForUUID2 theRelatedPATH2\n
someArbitraryCharactersForUUID3 theRelatedPATH3\n
...
...
```

then the following python script ```get_quarantined.py``` will help executing a command with args reading from the txt file
```python
import os
import glob
import argparse

def retrieve_files(txtfile):
    f = open(txtfile,"r")
    lines = f.readlines()
    for line in lines:
        uuid = line.split(" ")[0]
        path = line.split(" ")[1][:-1]
        os.system("nci-file-expiry recover {} '{}'".format(uuid, path))
    f.close()

if __name__ == "__main__":
    parser = argparse.ArgumentParser(usage=__doc__)
    parser.add_argument('-txtfile', '--txtfile',       required=True, type=str)

    args=parser.parse_args()
    retrieve_files(args.txtfile)

```

- Submit multiple task using nci-parallel

The executation of the above code use one core to deal will one single txt file. If we have ~ 100 txt files, we need to have quite a lot of jobs submitted. Each job will occupy a node while only use one core to process the task, leading to inefficency of the HPC computing. To avoid this, we can use the [nci-parallel module](https://opus.nci.org.au/display/Help/nci-parallel) to submit the job. Particularly, we firstly create a text file to include all the command lines that we want to run, one task per line. In this work flow, this file ```cmds.txt``` can be generated with the following bash script.

```bash
for file in x*;
do
    echo "python3 get_quarantined.py --txtfile $file" >>cmds.txt
done
```
where x* are the files splitted by the large quarantined_list.txt file.

With the following script, we can then get all files in the ```quarantined_list.txt``` back from quarantined.

```bash
#!/bin/bash
#PBS -P w47
#PBS -N test_parallel
#PBS -q normal
#PBS -l walltime=05:00:00
#PBS -l ncpus=144,mem=500GB
#PBS -l storage=scratch/a00+scratch/b00

#PBS -l wd

module load nci-parallel/1.0.0a
export ncores_per_task=1
export ncores_per_numanode=12
module load python3
ulimit -s unlimited
cd $PBS_O_WORKDIR

mpirun -np $((PBS_NCPUS/ncores_per_task)) --map-by ppr:$((ncores_per_numanode/ncores_per_task)):NUMA:PE=${ncores_per_task} nci-parallel --input-file cmds.txt --timeout 10000
```

