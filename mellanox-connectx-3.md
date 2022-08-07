# Download latest Mellanox Firmware tools and install them  + dependencies:  
sudo apt-get install gcc make dkms unzip linux-headers-$(uname -r)  
wget https://www.mellanox.com/downloads/MFT/mft-4.21.0-99-x86_64-deb.tgz  
tar -xvf mft-4.21.0-99-x86_64-deb.tgz && cd mft-4.21.0-99-x86_64-deb  
sudo ./install.sh

# Start MST service:
mst start
mst status
# copy the dev address with cr0 in it, like:
/dev/mst/mt4099_pci_cr0
# Use that address in the following commands

# Download latest FCBT firmware and unzip:
wget http://www.mellanox.com/downloads/firmware/fw-ConnectX3-rel-2_42_5000-MCX354A-FCB_A2-A5-FlexBoot-3.4.752.bin.zip
unzip fw-ConnectX3-rel-2_42_5000-MCX354A-FCB_A2-A5-FlexBoot-3.4.752.bin.zip

# Backup the cards current board definition file just in case
flint -d /dev/mst/mt4099_pci_cr0 dc orig_firmware.ini

# Flash the FCBT image to the card
flint -d /dev/mst/mt4099_pci_cr0 -i fw-ConnectX3-rel-2_42_5000-MCX354A-FCB_A2-A5-FlexBoot-3.4.752.bin -allow_psid_change burn

Cold boot the server when done

# Some useful config commands to config the card when it comes back up:

Code:

#start mst service again
mst start

#get detailed firmware info
mlxfwmanager --query

#get the device ID again:
mst status

#use it to see the cards current configuration:
mlxconfig -d /dev/mst/mt4099_pci_cr0 query

use it in config commands to configure the card
#for instance, to turn both ports from VPI/Auto to Ethernet only:
mlxconfig -d /dev/mst/mt4099_pci_cr0 set LINK_TYPE_P1=2 LINK_TYPE_P2=2


#turn off bootrom crap
mlxconfig -d /dev/mst/mt4099_pci_cr0 set BOOT_OPTION_ROM_EN_P1=false
mlxconfig -d /dev/mst/mt4099_pci_cr0 set BOOT_OPTION_ROM_EN_P2=false
mlxconfig -d /dev/mst/mt4099_pci_cr0 set LEGACY_BOOT_PROTOCOL_P1=0
mlxconfig -d /dev/mst/mt4099_pci_cr0 set LEGACY_BOOT_PROTOCOL_P2=0


##optional: delete bootrom off the card, so it doesn't slow down boot by popping up crap
##this is safe to do and supported by mellanox
flint -d /dev/mst/mt4099_pci_cr0 --allow_rom_change drom

