# DUAL-BOOT_IOS15-14
thanks dualbootfun 
https://dualbootfun.github.io/dualboot/index.html forked to ios 15 xd 

firstly please try to read all thing in https://dualbootfun.github.io/dualboot/index.html to understand how that work after you can read that 

how to dualboot ios 15.7 with ios 14.3 only people that know a lot about it 

you can do it only in macos but you can do it in linux however i had a issue that was that command asr -source xxx.xxxxx.xxx.dmg -target out.dmg --embed -erase -noprompt --chunkchecksum --puppetstrings because in linux you can't use that or i don't know how 
so I did that in a iphone 6s if you have other device, could be different do it. iphone or ipad with a10 please NOT use password. 

program that you need to download

1: irecovery, you can find that in linux and macos.
2: img4, img4tool.
3: iBoot64Patcher, you can find for linux and macos into https://nightly.link/Cryptiiiic/iBoot64Patcher/workflows/ci/main
4: Kernel64Patcher, git clone https://github.com/iSuns9/Kernel64Patcher.git | cd Kernel64Patcher | gcc Kernel64Patcher.c -o Kernel64Patcher
5: ldid, https://github.com/ProcursusTeam/ldid/releases/tag/v2.1.5-procursus5
6: python3 
7: devicetreepatcher "git clone https://github.com/Ralph0045/dtree_patcher"
8: gaster "git clone https://github.com/0x7ff/gaster"
start

you have to download your ipsw for any version of 14.0-14.8 for your device.

download https://github.com/verygenericname/SSHRD_Script and boot your device with ssh ramdisk. when you have access to ssh, execute ./sshrd.sh dump-blobs in order to dump shsh.

1: execute "img4tool -e -s *.shsh2 -m IM4M", with the shsh already dumped.

2: "./sshrd.sh ssh" to conect with ssh when that is alrady conect execute "ls /dev/" so you will see something like this disk0s1s* note the last nummber where is * so execute this command "newfs_apfs -o role=r -A -v SystemB /dev/disk0s1" that will be the system partition so if you execute again "ls /dev/" you would see the last nummber change in my case was disk0s1s8, execute this command "newfs_apfs -o role=0 -A -v DataB /dev/disk0s1" in my case that was disk0s1s9.

3: execute the partitions were created "mount_apfs /dev/disk0s1s* /mnt*" in my case "disk0s1s8 /mnt8" and "mount_apfs /dev/disk0s1s9 /mnt9"

4 : The root filesystem is always the largest file with a ".dmg" extension. The two smaller .dmg files are the update and restore ramdisks. It should be noted that rootfs names across different iOS versions vary.
so in your mac execute "asr -source xxx.xxxxx.xxx.dmg -target out.dmg --embed -erase -noprompt --chunkchecksum --puppetstrings"
the result of that will be out.dmg that you will copy into /mnt8 for ssh execute "scp -P 2222 out.dmg root@localhost:/mnt8". 
after execute "cp -av /mnt2/keybags /mnt9/" and "cp -av /mnt8/private/var/* /mnt9/"

5: reboot your device and execute sshrd.sh boot to boot ssh ramdisk after that execute in your iphone with ssh "/System/Library/Filesystems/apfs.fs/apfs_invert -d /dev/disk0s1 -s 8 -n out.dmg" change 8 with your systemb partition.

pathing boot
6: img4 -i iBSS.* -o iBSS.dec -k iv_key, img4 -i iBEC.* -o iBEC.dec -k iv_key

"iBoot64Patcher iBSS.dec iBSS.patched"

"img4 -i iBSS.patched -o iBSS.img4 -M IM4M -A -T ibss" im4m is dumped shsh that you have 

"iBoot64Patcher iBEC.dec iBEC.patched -n -b "rd=disk0s1s8 -v"" change 8 with your systemb partition 

"img4 -i iBEC.patched -o iBEC.img4 -M IM4M -A -T ibec"

using some hex editor Search for "com.apple.kernelcache" and change the 0 behind it to 5 (or your System partition-1, disk0s1s1=0, disk0s1s2=1) so in my case disk0s1s8=7

patching amfi:

"img4 -i kernelcache.* -o kcache.raw"

"Kernel64Patcher kcache.raw kcache.patched -a"

"touch kc.bpatch"
use "git clone https://github.com/mcg29/kerneldiff" to compare the kernel so execute "python3 kerneldiff.py kcache.raw kcache.patched" 
"img4 -i kernelcache.release.* -o kernelcache.img4 -M IM4M -T rkrn -P kc.bpatch"

please patch your devicetree using "img4 -i DeviceTree* -o dtree.raw" | "dtree_patcher dtree.raw dtree.patched -d" | "img4 -i dtree.patched -o devicetree.img4 -A -M IM4M -T rdtr"

patch trustcache 
"img4 -i xxx.xxxx.xxx.dmg.trustcache -o trustcache.img4 -M IM4M -T rtsc" 

Pack aopfw, ISP (adc-nike), AudioCodecFirmware (CallanFirmware), Multitouch and Trustcache into img4:
img4 -i image.im4p -o image.img4 -M IM4M

so you have already all thing so boot you device with ssh ramdisk sshrd.sh boot so when you are conected by ssh execute "mount_filesystems" after "mount_apfs /dev/disk0s1s7 /mnt3", "mount_apfs /dev/disk0s1s4 /mnt4", "mount_apfs /dev/disk0s1s5 /mnt5" so in your computer execute "scp -P2222 root@localhosts:/mnt3-4-5-6 ." with each number that you see that i do to backup the preboot and other things. execute "mount_apfs /dev/disk0s1s8 " 

 after reboot and put in mode dfu your device to exploit with gaster execute "gaster pwn" after that execute 
 
 booting with the patchers  
 
 irecovery -f iBSS.img4

irecovery -f iBECRamdisk.img4

Device screen should go blank and light up again once iBEC boots.

send and load ramdisk:
irecovery -f ramdisk

irecovery -c ramdisk

send and load devicetree:
irecovery -f devicetree.img4

irecovery -c devicetree

irecovery -f trustcache.img4

irecovery -c firmware

Send kernelcache and bootx:
irecovery -f kernelcache.img4
irecovery -c bootx

recorded the screen of your iphone because that will show you a name of directory preboot see the video and note the name of the directory so boot your ssh ramdisk again execute "mount_filesystems", "mkdir /mnt6/ + "the name of the preboot directory"" after boot again you should see booting your device 



thanks all person that 
thanks to everyone involved on tool and all 

thanks https://dualbootfun.github.io/dualboot/index.html that was that gave me the idea to dualboot my iphone 

something that if a person know to fix please tell us how : 
1: taurine edit to jump amfi because is already patched by kernel64patcher and others problem 
2: each time that you boot both your first system or your second system elimineted the others preboot directories of the other system

