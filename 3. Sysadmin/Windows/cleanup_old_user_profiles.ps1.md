# Clean up old user profile directories and files based on a cutoff date
# Targets C:\Users, skips excluded patterns, removes dirs/files older than cutoff

$baseDirectory = "C:\Users"
$cutoffDate    = Get-Date "2023-01-01"  # Adjust cutoff date as needed

$excludedFolderPatterns = @("*TEMP*", "*admin*", "*Public*", "*OneDrive*")

function RemoveOldDirectories {
    param ([string]$directory)

    $subdirectories = Get-ChildItem -Path $directory -Directory -ErrorAction SilentlyContinue
    if (-not $subdirectories) { return }

    foreach ($subdir in $subdirectories) {
        $exclude = $false
        foreach ($pattern in $excludedFolderPatterns) {
            if ($subdir.Name -like $pattern) { $exclude = $true; break }
        }
        if ($exclude) {
            Write-Host "Excluding: $($subdir.FullName)"
            continue
        }
        if ($subdir.LastWriteTime -lt $cutoffDate) {
            Write-Host "Removing: $($subdir.FullName)"
            Remove-Item -Path $subdir.FullName -Recurse -Force
        } else {
            RemoveOldDirectories -directory $subdir.FullName
        }
    }
}

function RemoveOldFiles {
    param ([string]$directory)

    $foldersToCheck = @("Downloads", "Documents", "Videos", "Pictures")
    foreach ($folder in $foldersToCheck) {
        $folderPath = Join-Path -Path $directory -ChildPath $folder
        $files = Get-ChildItem -Path $folderPath -File -ErrorAction SilentlyContinue
        if (-not $files) { continue }

        foreach ($file in $files) {
            if ($file.LastAccessTime -lt $cutoffDate) {
                Write-Host "Removing: $($file.FullName)"
                Remove-Item -Path $file.FullName -Force
            }
        }
    }
}

$userDirectories = Get-ChildItem -Path $baseDirectory -Directory -ErrorAction SilentlyContinue
foreach ($userDir in $userDirectories) {
    RemoveOldDirectories -directory $userDir.FullName
}

$subDirectories = Get-ChildItem -Path $baseDirectory -Directory -Recurse -ErrorAction SilentlyContinue
foreach ($subDir in $subDirectories) {
    RemoveOldFiles -directory $subDir.FullName
}

Write-Host "Cleanup complete."