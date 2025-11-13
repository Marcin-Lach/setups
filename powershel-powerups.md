This is to be verified when configuring next machine
Setup gathered from Scott Hanselman's blog posts

```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
Set-ExecutionPolicy Bypass -Scope CurrentUser
Install-Module -Name PackageManagement -Repository PSGallery -Force
Install-Module -Name PowerShellGet -Repository PSGallery -Force
```


When completed successfully, start a new PowerShell prompt as admin, and install required modules from PowerShell command-line:

```
Set-ExecutionPolicy Bypass -Scope
Process -Force; [System.Net.ServicePointManager]::SecurityProtocol
 = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

Set-ExecutionPolicy Bypass -Scope CurrentUser
Install-Module z -AllowClobber
Install-Module posh-git -AllowClobber
Install-Module oh-my-posh -AllowClobber
Install-Module Terminal-Icons -AllowClobber
Install-Module PSReadLine -AllowPrerelease -Force
```

```
code $PROFILE
```

```
Import-Module z
Import-Module -Name posh-git
Import-Module -Name oh-my-posh
Import-Module -Name Terminal-Icons

# Chocolatey profile
$ChocolateyProfile = "$env:ChocolateyInstall\helpers\chocolateyProfile.psm1"
if (Test-Path($ChocolateyProfile)) {
  Import-Module "$ChocolateyProfile"
}

# PSReadLine
if ($host.Name -eq 'ConsoleHost')
{
    Import-Module PSReadLine
}

# Aliases
Set-Alias grep findstr -Option AllScope
Set-Alias add "git add" -Option AllScope
Set-Alias status "git status" -Option AllScope
Set-Alias commit "git commit" -Option AllScope
Set-Alias gut git -Option AllScope

# Helper function to change directory to my development workspace
# Change c:\ws to your usual workspace and everytime you type
# in cws from PowerShell it will take you directly there.
function cws { Set-Location c:\repos }

# Helper function to set location to the User Profile directory
function cuserprofile { Set-Location ~ }
Set-Alias ~ cuserprofile -Option AllScope

# Helper function to show Unicode character
function U
{
    param
    (
        [int] $Code
    )
 
    if ((0 -le $Code) -and ($Code -le 0xFFFF))
    {
        return [char] $Code
    }
 
    if ((0x10000 -le $Code) -and ($Code -le 0x10FFFF))
    {
        return [char]::ConvertFromUtf32($Code)
    }
 
    throw "Invalid character code $Code"
}

# Default the prompt to agnoster oh-my-posh theme
# Set-PoshPrompt -Theme slim
Set-PoshPrompt -Theme ~\.mytheme.omp.json

# PSReadLine setup

Set-PSReadLineOption -PredictionSource History
#Set-PSReadLineOption -PredictionViewStyle ListView
Set-PSReadLineOption -EditMode Windows

Set-PSReadLineKeyHandler -Chord "Ctrl+f" -Function ForwardWord
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward

Set-PSReadLineKeyHandler -Key Ctrl+UpArrow -Function PreviousSuggestion
Set-PSReadLineKeyHandler -Key Ctrl+DownArrow -Function NextSuggestion

Set-PSReadLineKeyHandler -Key Ctrl+Shift+b `
                         -BriefDescription BuildCurrentDirectory `
                         -LongDescription "Build the current directory" `
                         -ScriptBlock {
    [Microsoft.PowerShell.PSConsoleReadLine]::RevertLine()
    [Microsoft.PowerShell.PSConsoleReadLine]::Insert("dotnet build")
    [Microsoft.PowerShell.PSConsoleReadLine]::AcceptLine()
}

Set-PSReadLineKeyHandler -Key Ctrl+Shift+r `
                         -BriefDescription ClearConsole `
                         -LongDescription "Clear the console window" `
                         -ScriptBlock {
    [Microsoft.PowerShell.PSConsoleReadLine]::RevertLine()
```
    [Microsoft.PowerShell.PSConsoleReadLine]::Insert("cls")
    [Microsoft.PowerShell.PSConsoleReadLine]::AcceptLine()
}
