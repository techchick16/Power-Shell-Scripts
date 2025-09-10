# Power-Shell-Scripts
Contains Power Shell scripts

# Extract-CertMetadata.ps1 file 
Extracts metadata from X.509 certificate files in a directory or single file.

Rename the certificate file(s) (usually sent to you as a .txt file) to end with .pem, .cer, or .crt. 
Create a new NotePad file and copy and paste the code below into it. 
Rename the C:\Path\To\Your\Script to a valid directory on your computer and where you plan to save out your Notepad file to.
Rename the C:\Path\To\Your\Script\MetaDataOutput.csv to a valid directory on your computer you want to save the meta data information out to in a .csv (comma delimited file).  
You need to have Excel installed on your computer to open the .csv file.
Save the file under C:\Path\To\Your\Script\Extract-CertMetadata.ps1.
Click on Windows button plus X at the same time and select Windows PowerShell.
This will take you to the command prompt, like an MS Dos command prompt. 
Type Cd "C:\Path\To\Your\Script" to go to the directory you put the Extract-CertMetadata.ps1 file in.
Type this command at the command prompt to run it: .\Extract-CertMetadata.ps1
This will produce a .csv file that should contain one row for each certificate found in the C:\Path\To\Your\Script directory.

<#
.SYNOPSIS
    Extracts metadata from X.509 certificate files in a directory or single file.

.PARAMETER Path
    "C:\Path\To\Your\Script\"

.PARAMETER OutputCsv
    "C:\Path\To\Your\Script\MetaDataOutput.csv"
#>

Param(
    [string]$Path = "C:\Path\To\Your\Script\",
    [string]$OutputCsv = "C:\Path\To\Your\Script\MetaDataOutput.csv"
)

function Get-CertMetadata {
    Param([string]$FilePath)

    try {
        $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($FilePath)
    }
    catch {
        Write-Warning "Cannot load certificate at $FilePath"
        return
    }

    [PSCustomObject]@{
        FileName            = [IO.Path]::GetFileName($FilePath)
        Subject             = $cert.Subject
        Issuer              = $cert.Issuer
        NotBefore           = $cert.NotBefore
        NotAfter            = $cert.NotAfter
        SerialNumber        = $cert.SerialNumber
        Thumbprint          = $cert.Thumbprint
        SignatureAlgorithm  = $cert.SignatureAlgorithm.FriendlyName
        PublicKeyAlgorithm  = $cert.PublicKey.Oid.FriendlyName
        PublicKeyLength     = $cert.PublicKey.Key.KeySize
        Extensions          = ($cert.Extensions | ForEach-Object {
                               $_.Oid.FriendlyName + ": " + $_.Format(0)
                             }) -join "; "
    }
}

# Determine certificate files (folder vs single file)
if (Test-Path $Path) {
    $item = Get-Item -Path $Path

    if ($item.PSIsContainer) {
        $certFiles = Get-ChildItem -Path $Path -Include *.cer,*.crt,*.pem -Recurse |
                     Where-Object { -not $_.PSIsContainer }
    }
    else {
        $certFiles = @($item)
    }
}
else {
    Write-Error "Path not found: $Path"
    exit 1
}

# Extract and display metadata
$results = $certFiles | ForEach-Object { Get-CertMetadata -FilePath $_.FullName }
$results | Format-Table -AutoSize

# Optional CSV export
if ($OutputCsv) {
    try {
        $results | Export-Csv -Path $OutputCsv -NoTypeInformation
        Write-Host "Metadata exported to $OutputCsv"
    }
    catch {
        Write-Warning "Failed to export CSV to $OutputCsv"
    }
}
