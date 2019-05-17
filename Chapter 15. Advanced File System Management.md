# Chapter 15. Advanced File System Management

# 1. Quota
# 2. Software RAID

## 2.1 What is RAID

**Redundant Arrays of Inexpensive Disks(RAID)**

### RAID-0(stripe): Most efficient

![RAID-0](http://cn.linux.vbird.org/linux_basic/0420quota_files/raid0.gif)

- All disks will be sliced into **equal blocks**.
- File will be cut into **equal parts**.
- All parts will be stored in blocks equally of disks.(That means it is fair for each disk)

Why efficient?
- Each disk deal with fewer data.(Distributed, all disks can work together to read a file)
- The total volume becomes bigger.

What is the demerit?
- Vulnerablity. If **any** block belonging to a file crashed, then the file can not be read!
- The volume of each disk had better to be the same to achieve more efficiency.(For example, if the small disk has been run out of, then the data will all been stored to the big ones.)

### RAID-1(mirror): Complete Backup

![RAID-1](http://cn.linux.vbird.org/linux_basic/0420quota_files/raid1.gif)

- Each part of the file will be copied to each disk equally.
- The volume is determined by the smallest one.
- The efficiency becomes lower when use **software RAID** writing because we only has one south-bridge.
- The efficiency becomes better when reading because RAID will balance the reading speed among processes.
- The safey is the same with the number of disks.

### RAID 0+1

![RAID 0+1](http://cn.linux.vbird.org/linux_basic/0420quota_files/raid01.gif)

- First, group disks and form RAID 0 to achieve efficiency.
- Second, group the RAID-0 groups to achieve safety.
- The file is safe if at least one group is alive.
- The volume is determined by the volume of a group.
- RAID 1+0 is similiar.

### RAID 5(balance between safety and efficiency.

![RAID 5](http://cn.linux.vbird.org/linux_basic/0420quota_files/raid5.gif)

- Each part of data is divided into two blocks. And the **parity** of the two blocks will be calculated.
- The two blocks and the parity will be stored in three disks fairly.
- The file can be restored when **any only one** disk crashed by the parity and one of the block.
- Reading efficiency is almost the same with RAID-0, but writing may be a little worse because it takes time to calculate parity.
- Volume is the number of disks minus one.
- RAID-6 uses two disks to store parity. Safety + 2 disks while volume - 2 disks.

### Spare Disk
- Connected to RAID but not used.
- When some disks in RAID crashed, the spare disks will be added to RAID automatically.
- You do not have to shut down and exchange disks.

### Merit
- Data safety and reliability
- Read/Writing efficiency.
- Volume

## 2.2 Software/Hardware RAID

## 2.3 Software RAID configuration 

## 2.4 RAID Rescue

## 2.5 Turn on auto-RAID and auto mount

## 2.6 Turn off RAID

# 3. Logical Volume Manger(LVM)
