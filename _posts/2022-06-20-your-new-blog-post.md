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
