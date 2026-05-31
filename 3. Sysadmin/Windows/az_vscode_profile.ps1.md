# -------------------------------
# VS Code PowerShell Profile
# Purpose: Fast, clean, Azure-focused deep work
# -------------------------------

# --- Quality of life ---
$ErrorActionPreference = "Stop"

# --- Oh My Posh Theme ---
# Note: Requires Nerd Font + oh-my-posh installed
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\paradox.omp.json" | Invoke-Expression

# --- Az Module Import (lightweight) ---
$azModules = "Az.Accounts", "Az.Resources", "Az.Compute", "Az.Network"

foreach ($module in $azModules) {
    if (-not (Get-Module -ListAvailable -Name $module)) {
        Write-Host "WARNING: $module not found. Installing..." -ForegroundColor Yellow
        try {
            Install-Module $module -Scope CurrentUser -Force -ErrorAction Stop
        } catch {
            Write-Host "ERROR: Failed to install $module. Install manually with:" -ForegroundColor Red
            Write-Host "   Install-Module $module -Scope CurrentUser" -ForegroundColor Red
            continue
        }
    }
    if (-not (Get-Module -Name $module)) {
        Write-Host "Importing module: $module" -ForegroundColor Cyan
        Import-Module $module -ErrorAction Stop -DisableNameChecking
    }
}

# --- Optional Auto-login (toggle with $env:AUTO_AZ_LOGIN=1) ---
try {
    if ($env:AUTO_AZ_LOGIN -eq "1" -and -not (Get-AzContext)) {
        Write-Host "Starting Azure authentication..." -ForegroundColor Cyan
        Connect-AzAccount -UseDeviceAuthentication | Out-Null
        Write-Host "Connected to Azure." -ForegroundColor Green
    }
} catch {
    Write-Host "WARNING: Azure login failed. Run Connect-AzAccount manually." -ForegroundColor Red
}

# --- Default Subscription (optional) ---
if ($env:DEFAULT_SUB) {
    try {
        Set-AzContext -Subscription $env:DEFAULT_SUB -ErrorAction Stop
    } catch {
        Write-Host "WARNING: Failed to set subscription $env:DEFAULT_SUB" -ForegroundColor Red
    }
}

# --- Shortcuts / Aliases ---
Set-Alias nrg New-AzResourceGroup
Set-Alias nvm New-AzVM

# --- Custom Console Colors / Logging ---
if ($PSVersionTable.PSEdition -eq "Core" -and $PSVersionTable.PSVersion.Major -ge 7) {
    $PSStyle.Formatting.Error   = $PSStyle.Foreground.Red
    $PSStyle.Formatting.Warning = $PSStyle.Foreground.Yellow
    $PSStyle.Formatting.Verbose = $PSStyle.Foreground.Cyan
    $PSStyle.Formatting.Debug   = $PSStyle.Foreground.Magenta
    $PSStyle.Progress.Style     = "Continuous"

    function Write-Log {
        param(
            [Parameter(Mandatory)]
            [ValidateSet("Info","Warn","Error","Success")]
            [string]$Level,
            [Parameter(Mandatory)]
            [string]$Message
        )

        switch ($Level) {
            "Info"    { Write-Host "$($PSStyle.Foreground.Cyan)INFO: $Message$($PSStyle.Reset)" }
            "Warn"    { Write-Host "$($PSStyle.Foreground.Yellow)WARN: $Message$($PSStyle.Reset)" }
            "Error"   { Write-Host "$($PSStyle.Foreground.Red)ERROR: $Message$($PSStyle.Reset)" }
            "Success" { Write-Host "$($PSStyle.Foreground.Green)OK: $Message$($PSStyle.Reset)" }
        }
    }
}
else {
    function Write-Log {
        param(
            [Parameter(Mandatory)]
            [ValidateSet("Info","Warn","Error","Success")]
            [string]$Level,
            [Parameter(Mandatory)]
            [string]$Message
        )

        switch ($Level) {
            "Info"    { Write-Host "INFO: $Message" -ForegroundColor Cyan }
            "Warn"    { Write-Host "WARN: $Message" -ForegroundColor Yellow }
            "Error"   { Write-Host "ERROR: $Message" -ForegroundColor Red }
            "Success" { Write-Host "OK: $Message" -ForegroundColor Green }
        }
    }
}

# --- Function: Colored Az Context Listing ---
function Get-AzContextWithColor {
    try {
        Get-AzContext -ListAvailable | ForEach-Object {
            $context = $_
            $color   = if ($context.IsDefault) { 'Green' } else { 'White' }
            $marker  = if ($context.IsDefault) { '*' } else { ' ' }
            Write-Host "$marker Name: $($context.Name)" -ForegroundColor $color
            Write-Host "  Subscription: $($context.Subscription.Name)" -ForegroundColor $color
            Write-Host "  Tenant: $($context.Tenant.Name)" -ForegroundColor $color
            Write-Host "  Account: $($context.Account.Id)" -ForegroundColor $color
            Write-Host ""
        }
    } catch {
        Write-Log -Level Warn -Message "No Azure contexts available."
    }
}

# --- Function: Colorized Help Output ---
function Get-HelpWithColor {
    [CmdletBinding(DefaultParameterSetName='Default')]
    param(
        [Parameter(Position=0, Mandatory=$true)]
        [string]$Name,

        [switch]$Full,
        [switch]$Detailed,
        [switch]$Examples,
        [switch]$Online
    )

    $args = @($Name)
    if ($Full)     { $args += "-Full" }
    if ($Detailed) { $args += "-Detailed" }
    if ($Examples) { $args += "-Examples" }
    if ($Online)   { $args += "-Online" }

    try {
        Get-Help @args | ForEach-Object {
            $line = $_.ToString()

            switch -Regex ($line) {
                '^SYNTAX'        { Write-Host $line -ForegroundColor Cyan }
                '^PARAMETERS'    { Write-Host $line -ForegroundColor Magenta }
                '^EXAMPLE'       { Write-Host $line -ForegroundColor Yellow }
                '^REMARKS'       { Write-Host $line -ForegroundColor DarkGray }
                '^RELATED LINKS' { Write-Host $line -ForegroundColor Green }
                default          { Write-Host $line }
            }
        }
    } catch {
        Write-Log -Level Error -Message "Failed to get help for $Name"
    }
}