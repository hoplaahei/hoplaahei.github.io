---
layout: post
published: true
---

This guide assumes the reader:

- can use a console
- already has an LVM setup

Creating an LVM snapshot is useful as it makes a "freeze" of the filesystem, so that running write operations do not leave the filesystem in an inconsistent state. LVM requires the creation of a snapshot volume to store them, but the snapshots only start to take up space when changes happen, so it is fine to limit the snap partition to a few gigs.

Here are a few key points to remember when dealing with LVM snapshots:

- Snapshots slow down the system considerably whilst active, so use them temporarily to create backups, then remove them

- Although the snapshots only contain changes, they link to and behave like the full filesystem. This is useful, as it allows us to back them up with conventional methods, as if they were a full copy of the volume at that time

- dd'ing large drives can be slow, so treat this as a one time operation to get the backups. From then on, mount the backups and use an incremental tool like rsnapshot to keep the source and backup volumes in sync

- Remember that the snapshot volume must be large enough to hold all changes that happen to the source volume whilst the snapshot is in existence

To illustrate how to use LVM snapshots, imagine two disks: one is an SSD, the other an external hard drive.

The SSD is our OS installation. It has:

- a root volume, `lvolroot`
- a home volume, `lvolhome`
- a `VolGroupSSD` volume group that contains the two logical volumes

The external hard disk is split into two partitions:

- `/dev/sda1` is an LVM partition which has no volumes. We use this to temporarily create and store LVM snapshots, then delete them
- `/dev/sda2` is another LVM partition thst will contain backup volumes of the SSD volumes

One possible problem is that the disk we want to backup (the SSD here) already has the full space allocated to LVM partitions, with no room left for a snapshot volume. In that case, we will use our external hard disk instead (`/dev/sdb1`), to temporarily store the snapshots. We let our `VolGroupSSD` see this device and share its space:

```
vgextend VolGroupSSD /dev/sdb1
```
You could just as easily use a USB pen, as we only need it temporarily for the snapshots, which get deleted afterwards. We will also `reduce` (unextend) the `VolGroupSSD` from the external disk when finished, as there is no point leaving it active to wake up our hard disk uneccessarily.

Now we need to create the volumes that will store the snapshots:

```
lvdisplay -v /dev/VolGroupSSD/lvolroot

lvdisplay -v /dev/VolGroupSSD/lvolhome
```

Look at the `LV Size` for the `lvolroot` and `lvolhome` logical volumes and decide how much % of that to use for their corresponding snapshot volumes. If files change on these volumes a lot then allocate a higher percentage. I personally just stay on the extreme safe side and allocate the full size of the volume for the snapshots. You can do this by taking the `Current LE` number and using it in the next commands:

```
lvcreate -l 5120 -s /dev/VolGroupSSD/lvolroot -n lvolroot-snap /dev/sdb1
lvcreate -l 22717 -s /dev/VolGroupSSD/lvolhome -n lvolhome-snap /dev/sdb1
```
This uses the extent numbers found in `lvdisplay` to allocate the exact size of the source volumes to the backup volumes, that way we prevent the possibility of overflow from too many file changes, which would drop the snapshot volume and prevent a backup. 

The /dev/... parts of those lvcreate commands cause LVM to write on the external hard disk rather than the SSD. I doubt this is really necessary -- as by default it should write linearly straight to the free space/extents -- but I add it anyway to be sure. 

Now that our snapshot partitions are active they create a slowdown on the system, so only keep them enabled for the copying to the backup volumes, which it is time to create on `/dev/sdb2` of the external hard disk. First make the volume group for the external hard disk:

```
vgcreate VolGroupBackupDisk /dev/sdb2
```

And create the logical backup volumes in it:

```
lvcreate -l 22717 -n lvolhome-backup /dev/mapper/VolGroupBackupDisk
lvcreate -l 5120 -n lvolroot-backup /dev/mapper/VolGroupBackupDisk
```

Use the same size/extents as the source volumes e.g., lvolroot. If you used a smaller size for the snapshot volumes then go with the size of e.g., `lvolroot`, not `lvolroot-snap`.

Now run the full backup:

```
dd if=/dev/VolGroupSSD/lvolroot-snap of=/dev/VolGroupBackupDisk/lvolroot-backup
dd if=/dev/VolGroupSSD/lvolhome-snap of=/dev/VolGroupBackupDisk/lvolhome-backup
```

The backup is done, so we remove the snapshots:

```
lvremove /dev/VolGroupSSD/lvolhome-snap
lvremove /dev/VolGroupSSD/lvolroot-snap
```
And unextend the hard disk from the SSD volume group as we no longer need the extra space for snapshots:

```
vgreduce VolGroupSSD /dev/sdb1
```

Remember to also backup your boot partition if it lives outside LVM with e.g.,:

```
lvcreate -L 512 -n boot-backup VolGroupBackupDisk
dd if=/dev/sda1 of=/dev/VolGroupBackupDisk/lvolboot-backup
```

I've made a script (found at the end of this page) to make all the above steps easier. The script stays on the safe side by allocating the full size of the volume to be backed up to the snapshot volume, unless you pass it an extent number with `-e`. 

If there is no room for snapshots on the source device then pass the script a `-d /dev/sdXN` argument and it will automatically extend to another device (e.g., a hard disk or USB) to use the extra space there, then it will unextend (reduce) afterwards. 

I give no guarantees that the following script won't destroy your system, so check over it yourself and make a conventional backup first. Save this script as lvmbackup and execute with e.g., `sh lvmbackup -i MyVolGroup -o MyBackupVolGroup -l lvolroot -d /dev/sdb1` (where sdb1 is a large hard disk to store the snapshots).

Here is how I backup my system with the script:

```
lvmbackup -i VolGroupSSD -o VolGroupBackupDisk -l lvolroot -d /dev/sdb1
lvmbackup -i VolGroupSSD -o VolGroupBackupDisk -l lvolhome -d /dev/sdb1
```

Notice that I have two volume groups. Just as in the examples above, VolGroupSSD contains the OS install on my SSD. VolGroupBackupDisk is the backup volume on my external hard disk. The above makes a `lvolroot-snap` and `lvolhome-snap` on the backup hard disk volume group, and uses them to copy over to backup volumes on that disk, then it removes the snapshots and unextends the hard disk from the SSD storage. 

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