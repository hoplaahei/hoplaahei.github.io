---
layout: post
published: true
---

This guide assumes the reader knows how to:

- use a terminal
- partition a disk

Creating an LVM snapshot is useful as it makes a "freeze" of the filesystem so that things are in a consistent state before making a backup. LVM requires the creation of a snapshot volume to store them, but the snapshots only store the changed files in the overlying filesystem, so it is ok to limit the volume size to a few gigs.

The downside of LVM snapshots are that they slow down the system considerably whilst they remain on the system, so just use them temporarily to get a frozen state of the disk, treat them like a mounted disk and back up from them using more conventional methods, then remove them. 

I've made the below script to make a temporary snapshot of a volume, dd it over to a backup volume, and then remove the snapshot volume. The script will extend over to another volume when given an optional "snapshot device" argument, and then remove (reduce) the extended device after. This is useful if there isn't enough room for temporary snapshots on the origin device. It is ok to use a slow device such as USB as the snapshot device, as we only need it during the backup, then remove the snapshot volume afterwards. 

dd'ing large drives is slow, so treat this as a one time operation to get the backups where needed. Don't use this script to backup every time. Instead, mount the logical volumes already backed up as normal drives, and use an incremental backup tool like rsnapshot to only copy over the changed files.

Save this script as lvm-backup.sh and execute with ./lvm-backup.sh:

```bash
#!/bin/bash
# Argument = -sg source-group -dg dest-group -sv source-vol
#            -ed extended-device -v

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
   -d     Extended device
   -v     Verbose
EOF
}

SOURCE_GROUP=
DEST_GROUP=
SOURCE_VOL=
EXTENDED_DEV=
VERBOSE=
while getopts “hi:o:l:d:v” OPTION
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

extentNum=$(lvdisplay -v /dev/$SOURCE_GROUP/$SOURCE_VOL | grep "Current\ LE" | grep -o '[0-9]*')

lvcreate -l $extentNum -n ${SOURCE_VOL}-backup $DEST_GROUP

lvcreate -l $extentNum -s /dev/$SOURCE_GROUP/$SOURCE_VOL -n ${SOURCE_VOL}-snap $EXTENDED_DEV

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