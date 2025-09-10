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

