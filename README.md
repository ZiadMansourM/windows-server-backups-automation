# WinSCP for Scripting and Task Automation - [link](https://winscp.net/eng/docs/scripting)
FTP clients like WinSCP can be automated using Command Line Interface (CLI), which allows for the execution of a series of commands to automate repetitive tasks. In addition to CLI, WinSCP offers a scripting/console interface with many commands that can be typed in interactively or read from a script file. For simple tasks, using the scripting interface directly is recommended, while for more complex tasks, using the WinSCP .NET assembly is preferred.

The WinSCP .NET assembly, winscpnet.dll, is a .NET wrapper around WinSCP’s scripting interface that enables your code to connect to a remote machine and manipulate remote files over SFTP, FTP, WebDAV, S3, and SCP sessions from .NET languages such as C# and VB.NET, or from automation environments supporting .NET such as PowerShell, SQL Server Integration Services (SSIS), and Microsoft Azure WebSites and Functions. The assembly is also exposed to COM, which allows it to be used from a variety of other programming languages and development environments, including WSH-hosted active scripting languages like JScript and VBScript, Visual Basic for Applications (VBA), Perl, and Python.

> The library is primarily intended for advanced automation tasks on Microsoft Windows that require conditional processing, loops or other control structures for which the basic [scripting interface](https://winscp.net/eng/docs/scripting) is too limited.

## [🔗](https://winscp.net/eng/docs/library_powershell#example) Synchronizing Directories with .NET Assembly and PowerShell
This PowerShell script is a `well-tested and fully-working example` of how to use the WinSCP .NET assembly to synchronize directories between a local machine and a remote server using SFTP protocol. The script can be modified to suit different needs and can be run on a schedule using Windows Task Scheduler. We will store logs here `C:\logs\winscp.log`

```.ps1
 # https://winscp.net/eng/docs/library_powershell#example

# Log file path
$logFilePath = "C:\logs\winscp.log"

# Define synchronization options as hash table
$syncOptions = @{
    SynchronizationMode = [WinSCP.SynchronizationMode]::Remote
    LocalDirectory = "C:\Users\ziadh\Production-Database-Files\"
    RemoteDirectory = "/PostgreSQL-Database-Backups/"
    RemoveFiles = $false
    Mirror = $false
    Criteria = [WinSCP.SynchronizationCriteria]::Size
    TransferOptions = New-Object WinSCP.TransferOptions -Property @{
        TransferMode = [WinSCP.TransferMode]::Binary
    }
}

# Helper function to create log message
function CreateLogMessage($message) {
    $logMessage = "@[$(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')] $message"
    Add-Content -Path $logFilePath -Value $logMessage
}

try
{
    # Load WinSCP .NET assembly
    Add-Type -Path "C:\Program Files (x86)\WinSCP\WinSCPnet.dll"
 
    # Setup session options
    $sessionOptions = New-Object WinSCP.SessionOptions -Property @{
        Protocol = [WinSCP.Protocol]::Sftp
        HostName = "<HOST>"
        UserName = “<USERNAME>”
        Password = "<PASSWORD>"
        SshHostKeyFingerprint = "<SshHostKeyFingerprint>"
    }
 
    $session = New-Object WinSCP.Session
 
    try
    {
        # Connect
        $session.Open($sessionOptions)

        # Log start of synchronization
        CreateLogMessage "Synchronization started $($syncOptions.SynchronizationMode) $($syncOptions.LocalDirectory) $($syncOptions.RemoteDirectory) $($syncOptions.Criteria)"
 
        # Synchronize directories, ref: https://winscp.net/eng/docs/library_session_synchronizedirectories
        $transferResult =
            $session.SynchronizeDirectories(
                $syncOptions.SynchronizationMode, # Synchronization mode: Local, Remote and Both
                $syncOptions.LocalDirectory, # Full path to local directory.
                $syncOptions.RemoteDirectory, # Full path to remote directory.
                $syncOptions.RemoveFiles, # Remove files from destination that do not exist in source. "bool removeFiles" When set to true, deletes obsolete files. Cannot be used for SynchronizationMode.Both.
                $syncOptions.Mirror, # "bool mirror" When set to true, synchronizes in mirror mode (synchronizes also older files). Cannot be used for SynchronizationMode.Both. Defaults to false.
                $syncOptions.Criteria, # criteria for synchronization: None, Time, Size, Either. For SynchronizationMode.Both SynchronizationCriteria.Time can be used only.
                $syncOptions.TransferOptions # additional options: Transfer options. Defaults to null, what is equivalent to new TransferOptions().
            )
 
        # Throw on any error
        $transferResult.Check()
 
        # Print results
        foreach ($transfer in $transferResult.Downloads)
        {
            CreateLogMessage "Download of $($transfer.FileName) succeeded"
        }
 
        foreach ($transfer in $transferResult.Uploads)
        {
            CreateLogMessage "Upload of $($transfer.FileName) succeeded"
        }
    }
    finally
    {
        # Disconnect, clean up
        $session.Dispose()
        # Log session disposal time
        CreateLogMessage "Session disposed @$(Get-Date -Format 'H:mm:ss')."
    }
 
    exit 0
}
catch
{
    CreateLogMessage "Error: $($_.Exception.Message)"
    exit 1
}
```


### Example of `C:\logs\winscp.log`
You see now two runs. First, local and remote directory were in sync. Second, only one file needed to be uploaded.
```.log
@[2023-03-31 07:51:15] Synchronization started Remote C:\Users\ziadh\Production-Database-Files\ /PostgreSQL-Database-Backups/ Size
@[2023-03-31 07:51:15] Session disposed @7:51:15.
@[2023-03-31 07:51:33] Synchronization started Remote C:\Users\ziadh\Production-Database-Files\ /PostgreSQL-Database-Backups/ Size
@[2023-03-31 07:51:33] Upload of C:\Users\ziadh\Production-Database-Files\test-logging.txt succeeded
@[2023-03-31 07:51:33] Session disposed @7:51:33.
```
