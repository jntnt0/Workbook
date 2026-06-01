# Get the partition you want to shrink (C drive)
$partitionToShrink = Get-Partition -DiskNumber 0 -PartitionNumber 1

if ($partitionToShrink) {
    # Calculate the new size for the partition (for example, shrink by 1 GB)
    $newSize = $partitionToShrink.Size - 1GB

    # Shrink the partition
    Resize-Partition -InputObject $partitionToShrink -Size $newSize

    # Get the partition you want to extend (Recovery partition)
    $partitionToExtend = Get-Partition -DiskNumber 0 -PartitionNumber 2

    if ($partitionToExtend) {
        # Check available space on the disk
        $disk = $partitionToExtend | Get-Disk
        $availableSpace = $disk.Size - ($disk | Get-Partition | Measure-Object -Property Size -Sum).Sum

        # Check if there is enough available space to extend the partition
        if ($availableSpace -ge 1GB) {
            # Extend the partition by 1 GB
            Resize-Partition -InputObject $partitionToExtend -Size ($partitionToExtend.Size + 1GB)
        } else {
            Write-Host "Not enough unallocated space available on the disk to extend the partition."
        }
    } else {
        Write-Host "Recovery partition not found."
    }
} else {
    Write-Host "C drive partition not found."
}
