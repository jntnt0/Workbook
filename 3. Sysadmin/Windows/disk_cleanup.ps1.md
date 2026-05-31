# Function to cleanup files in a directory
function CleanupFiles {
    param(
        [string]$Path,
        [string]$FileType
    )

    $cleanupResults = @()

    try {
        $files = Get-ChildItem -Path $Path -File -ErrorAction Stop

        foreach ($file in $files) {
            try {
                Remove-Item -Path $file.FullName -Force -ErrorAction Stop
                Write-Verbose "Deleted $($file.FullName)"
                $cleanupResults += [PSCustomObject]@{
                    "Status" = "Success"
                    "File" = $file.Name
                    "FileType" = $FileType
                }
            } catch {
                $cleanupResults += [PSCustomObject]@{
                    "Status" = "Failed"
                    "File" = $file.Name
                    "Error" = $_.Exception.Message
                    "FileType" = $FileType
                }
            }
        }
    } catch {
        $cleanupResults += [PSCustomObject]@{
            "Status" = "Failed"
            "File" = "N/A"
            "Error" = $_.Exception.Message
            "FileType" = $FileType
        }
    }

    return $cleanupResults
}

# Function to empty Recycle Bin
function CleanupRecycleBin {
    param()

    $cleanupResults = @()

    try {
        Clear-RecycleBin -Force -ErrorAction Stop
        Write-Verbose "Recycle Bin emptied"
        $cleanupResults += [PSCustomObject]@{
            "Status" = "Success"
            "File" = "Recycle Bin"
            "FileType" = "Recycle Bin"
        }
    } catch {
        $cleanupResults += [PSCustomObject]@{
            "Status" = "Failed"
            "File" = "Recycle Bin"
            "Error" = $_.Exception.Message
            "FileType" = "Recycle Bin"
        }
    }

    return $cleanupResults
}

# Perform disk cleanup with progress reporting
Write-Output "Starting disk cleanup..."

# Delete temporary files in the user's temp directory and track success/failure (ignore errors)
$cleanupResults = @()
$tempDir = [System.IO.Path]::GetTempPath()
Write-Output "Cleaning up temporary files in $tempDir..."
$cleanupResults += CleanupFiles -Path $tempDir -FileType "Temporary Files"

# Clean up system files
Write-Output "Cleaning up system files..."
$cleanupResults += CleanupFiles -Path $env:SystemRoot -FileType "System Files"
$cleanupResults += CleanupFiles -Path "$env:SystemRoot\SoftwareDistribution\Download" -FileType "Windows Update Files"
$cleanupResults += CleanupFiles -Path "$env:SystemRoot\Logs" -FileType "Logs"
$cleanupResults += CleanupFiles -Path "$env:SystemRoot\Temp" -FileType "Temp Files"

# Empty the Recycle Bin
Write-Output "Emptying Recycle Bin..."
$cleanupResults += CleanupRecycleBin

# Clean up Windows Error Reporting files
Write-Output "Cleaning up Windows Error Reporting files..."
$cleanupResults += CleanupFiles -Path "$env:SystemRoot\Windows\WER" -FileType "Windows Error Reporting Files"

# Clean up Delivery Optimization files
Write-Output "Cleaning up Delivery Optimization files..."
$cleanupResults += CleanupFiles -Path "$env:SystemRoot\SoftwareDistribution\DeliveryOptimization" -FileType "Delivery Optimization Files"

# Clean up DirectX Shader Cache
Write-Output "Cleaning up DirectX Shader Cache..."
$cleanupResults += CleanupFiles -Path "$env:USERPROFILE\AppData\Local\Microsoft\Windows\INetCache\ShaderCache" -FileType "DirectX Shader Cache"

# Clean up thumbnail cache
Write-Output "Cleaning up thumbnail cache..."
$cleanupResults += CleanupFiles -Path "$env:USERPROFILE\AppData\Local\Microsoft\Windows\Explorer" -FileType "Thumbnail Cache"

# Count successful and failed deletions
$successfulDeletions = ($cleanupResults | Where-Object { $_.Status -eq "Success" }).Count
$failedDeletions = ($cleanupResults | Where-Object { $_.Status -eq "Failed" }).Count

# Display summary of deletions
Write-Output "Disk cleanup complete."
Write-Output "Successfully deleted: $successfulDeletions files"

if ($failedDeletions -gt 0) {
    Write-Warning "Failed to delete: $failedDeletions files."
}




