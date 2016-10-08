#Scrolling Through Console Output

`Shift` + `PgUp` or `PgDn` will scroll through the console log.

#Securely wiping a disk

    dd if=/dev/urandom of=/dev/sdX bs=512

- Find out the `bs` value by running `fdisk -l /dev/sdX` and look for the "physical sector size".
- To check on the progress of `dd`, run the following command **in a separate terminal**:
        
        kill -USR1 $(pgrep dd)

#Getting the UUIDs of Partitions
    
    lsblk -no name,uuid
    
