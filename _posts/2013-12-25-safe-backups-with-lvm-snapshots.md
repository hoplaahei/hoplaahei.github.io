---
layout: post
published: true
---

This guide assumes the reader knows how to:

- use a terminal
- partition a disk

Creating an LVM snapshot is useful as it makes a "freeze" of the filesystem so that running write operations do not leave the filesystem in an inconsistent state. LVM requires the creation of a snapshot volume to store them, but the snapshots only start to take up space when changes happen, so it is fine to limit the snap partition to a few gigs.

Here are a few key points to remember when dealing with LVM snapshots:

- Snapshots slow down the system considerably whilst active, so use them temporarily to create backups, then remove them

- Although the snapshots only contain information about changes, they link back and behave like the full filesystem when copied over. This is useful, as it allows backup using more conventional methods

- dd'ing large drives can be slow, so treat this as a one time operation to get the backups. From then on, mount the backups and use an incremental tool like rsnapshot to keep the source and backup volumes in sync

- Remember that the snapshot volume must be large enough to hold all changes that happen to the source volume while the snapshot is in existence

I've made a script (found at the end of this page) to make a temporary snapshot of a volume A, dd it over to a backup volume B (making a full backup of the volume A), and then remove the snapshot volume. My script stays on the safe side by allocating the full size of the volume to be backed up to the snapshot volume, unless you specifically pass it an extent number with `-e`. 

If there is no room for snapshots on the source device then pass the script a `-d /dev/sdXN` argument and it will automatically extend to another device to use the extra space there, then unextend (reduce) afterwards. Using a slow device such as USB pen as the snapshot device is ok, as we only need it during the backup and can simply remove the snapshot volume afterwards. 

I give no guarantees that the following script won't destroy your system, so check over it yourself and make a conventional backup first. Save this script as lvmbackup and execute with e.g., `sh lvmbackup -i MyVolGroup -o MyBackupVolGroup -l lvolroot -d /dev/sdb1` (where sdb1 is a large hard disk to store the snapshots).

Here is how I backup my system with the script:

```
lvmbackup -i VolGroupSSD -o VolGroupBackupDisk -l lvolroot -d /dev/sda1
lvmbackup -i VolGroupSSD -o VolGroupBackupDisk -l lvolhome -d /dev/sda1
```

Notice that I have two volume groups. VolGroupSSD contains the OS install on my SSD. VolGroupBackupDisk is the backup volume on my external hard disk. The above makes a `lvolroot-snap` and `lvolhome-snap` on the backup hard disk volume group, and uses them to copy over to backup volumes on that disk, then it removes the snapshots. 

Remember to also backup your boot partition if it lives outside LVM with e.g.,:

```
lvcreate -L 512 -n boot-backup VolGroupBackupDisk
dd if=/dev/sdb1 of=/dev/VolGroupBackupDisk/lvolboot-backup
```
Here is the script itself:

```bash
#!/bin/bash
# Argument = -i source-group -o dest-group -l source-vol
#            -d extended-snapshot-device -e extent-num -v

usage()
{
cat <<EOF
usage: $0 options

This script backs up logical volumes using snapshots.

OPTIONS:
   -h     Show this messagea
   -i     Source Group Volume
   -o     Destination Group Volume
   -l     Source Logical Volume
   -d     Extended snapshot device (optional)
   -e     Extent number (optional)
   -v     Verbose
EOF
}

SOURCE_GROUP=
DEST_GROUP=
SOURCE_VOL=
EXTENDED_DEV=
EXTENT_NUM=
VERBOSE=
while getopts “hi:o:l:d:e:v” OPTION
do
     case $OPTION in
         h)
             usage
             exit 1
             ;;
         i)
             SOURCE_GROUP=$OPTARG
             ;;
         o)
             DEST_GROUP=$OPTARG
             ;;
         l)
             SOURCE_VOL=$OPTARG
             ;;
         d)
             EXTENDED_DEV=$OPTARG
             ;;
         e)
             EXTENT_NUM=$OPTARG
             ;;
         v)
             VERBOSE=1
             ;;
         ?)
             usage
             exit
             ;;
     esac
done

if [[ -z $SOURCE_GROUP ]] || [[ -z $DEST_GROUP ]] || [[ -z $SOURCE_VOL ]]
then
     usage
     exit 1
fi

if [ ! -z "$EXTENDED_DEV" ];
then
    vgextend $SOURCE_GROUP $EXTENDED_DEV
else
    echo "WARNING: About to use one device. Make sure it has enough space." && sleep 5
fi

if [ -z "$EXTENT_NUM" ];
then
    EXTENT_NUM=$(lvdisplay -v /dev/$SOURCE_GROUP/$SOURCE_VOL | grep "Current\ LE" | grep -o '[0-9]*')
fi

lvcreate -l $EXTENT_NUM -n ${SOURCE_VOL}-backup $DEST_GROUP

lvcreate -l $EXTENT_NUM -s /dev/$SOURCE_GROUP/$SOURCE_VOL -n ${SOURCE_VOL}-snap $EXTENDED_DEV

dd if=/dev/$SOURCE_GROUP/${SOURCE_VOL}-snap of=/dev/$DEST_GROUP/${SOURCE_VOL}-backup

echo
echo Cleaning up....
echo

lvremove /dev/$SOURCE_GROUP/${SOURCE_VOL}-snap

if [[ ! -z $EXTENDED_DEV ]];
then
    vgreduce $SOURCE_GROUP $EXTENDED_DEV
fi
```