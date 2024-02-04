# Lessons learnt from bootloader setup (systemd)


* first check what is the partition that is configured for booting (esp = EFI System Partition) (e.g. nvme0n1p3), if you have a dual boot scheme there will probably exist two separate partitions:
	* run 
```
lsblk -o name,size,fstype,parttype,mountpoints
```
fstype should be vfat and parttype should be `c12a7328-f81f-11d2-ba4b-00a0c93ec93b`
* check if the partition you're interested in has a mount point, specifying the directory this partition is accessible from the user
* if there's a mount point --> this means: no `bootx64.efi` file or located in wrong directory
	* you need to add/move/copy the `bootx64.efi` file there
		* if `bootx64.efi` doesn't exist in your pc, search for `systemd-bootx64.efi` instead (probably loacted in `/usr/lib/systemd/boot/efi/systemd-bootx64.efi`), copy to a path realtive to the mount point and rename it to `bootx64.efi`
	* run 
	```
		sudo bootctl --path=/path/you/copied/bootx64.efi/to install
	```
	* Notes: you can copy Pop OS's default configuration (/boot/efi) and make the file's `bootx64.efi` path: /mount/point/boot/efi, so the last command becomes: `sudo bootctl -esp--path=/mount/point/boot/efi install`
* if there's no mount point indicated --> `bootx64.efi` doesn't have access to the esp and additionally there may be no such file
	* you need to give to a directory you can access and edit (preferably the root) access to this partition
		* make the directory `/boot/efi` if it doesn't exist
		* run
		```
		mount dev/esp_partition_name mount/point/boot/efi
		```
		* access the directory `mount/point/boot/efi` and add/move/copy the `bootx64.efi` file as described above
		* run
		```
		 sudo bootctl --path=mount/point/boot/efi install
		```
		* Note: since systemd-boot will try to find the esp in /boot, /efi and /boot/efi by default, the last command can be altered to: `sudo bootctl install`
#### Notes:
* If you have a dual boot scheme and the Pop OS entry isn't showing, you need to boot from a live USB to gain access to a terminal, and follow the instructions above though the mountpoints now point to the `/media` folder, except for the partition's path which will rename `dev/partition_name`. So for the rest of the paths, the correct parent path (corresponding to the root partition) should be added before the paths that are shown above.
