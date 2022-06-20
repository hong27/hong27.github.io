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

then the following python script will help executing a command with args reading from the txt file
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




#### Some T-SQL Code

```tsql
SELECT This, [Is], A, Code, Block -- Using SSMS style syntax highlighting
    , REVERSE('abc')
FROM dbo.SomeTable s
    CROSS JOIN dbo.OtherTable o;
```

#### Some PowerShell Code

```powershell
Write-Host "This is a powershell Code block";

# There are many other languages you can use, but the style has to be loaded first

ForEach ($thing in $things) {
    Write-Output "It highlights it using the GitHub style"
}
```
