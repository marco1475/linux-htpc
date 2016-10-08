#Bootable Flash Drive

1. Download [Easy2Boot](http://www.easy2boot.com/download/).
2. Extract *Easy2Boot* to a new, empty directory.
3. Run the `Make_E2B_USB_DRIVE (run as admin).cmd` script as administrator to create a bootable flash drive.
4. Copy [UEFI-bootable](#uefi---bootable-images) images to the `_ISO/MAINMENU` directory.
    - Optionally create a human-readable entry in the boot menu by dragging-and-dropping the `.imgPTN` file onto the `_ISO/docs/E2B Utilities/E2B TXT Maker.cmd` script.
5. Ensure the flash drive is contiguous by running the `MAKE_THIS_DRIVE_CONTIGUOUS.cmd` script.

#UEFI-bootable Images

1. Download [MPI Toolkit](http://www.easy2boot.com/download/mpi-pack/).
2. Extract *MPI Toolkit* to a new, empty directory.
3. Install *ImDisk* from the `ImDisk` directory.
4. Run the `CreateDesktopShortcuts.cmd` script.
5. Convert the ISOs into UEFI-bootable `.imgPTN` images by drag-and-dropping them onto the `MPI_FAT32` Desktop shortcut.


