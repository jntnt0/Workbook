# Link a Private DNS Zone to a VNet with auto-registration enabled
$ResourceGroup  = "your-rg"
$ZoneName       = "your-zone.local"
$LinkName       = "link-to-your-vnet"
$VNetName       = "your-vnet"

New-AzPrivateDnsVirtualNetworkLink `
    -ResourceGroupName $ResourceGroup `
    -ZoneName $ZoneName `
    -Name $LinkName `
    -VirtualNetworkId (Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroup).Id `
    -EnableRegistration:$true