## Install and Configure
I used [Raspberry Pi imager](https://www.raspberrypi.org/software/) to install the Ubuntu 20.10 64bit on each SD Card.  Insert these into the Pi's and power them on.

The image is configured with DHCP client, [Pi device MAC addresses are prefixed DC:A6:32](https://maclookup.app/macaddress/DCA632). I connected to my router which acts as DHCP server and found the four leases sorting by MAC. With the DHCP addresses can connect via SSH, the Ubuntu image has default username of `ubuntu` and password `ubuntu`. You're prompted to change password at first connect.

![Ubuntu Password](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/ubuntu-pw.png)

I want to reliably know how to connect to these and like to change from dynamic to a staticly asssigned IP address. To do this for Ubuntu 20.10 we update Netplan configuration.

```
sudo vi /etc/netplan/50-cloud-init.yaml
```

Here is example of how I update this to reflect static IP.

```
network:
    ethernets:
        eth0:
            addresses: [192.168.1.100/24]
            gateway4: 192.168.1.254
            nameservers:
              addresses: [8.8.8.8,8.8.4.4]
            dhcp4: no
            match:
                driver: bcmgenet smsc95xx lan78xx
            optional: true
            set-name: eth0
    version: 2
```

With the configuration file updated can have netplan load the config.

```
sudo netplan --debug apply
```

With any new install its useful to apply latest patches.

```
sudo apt update
sudo apt upgrade -y
```

## Install Tools

The VideoCore packages provide command line utilities that can get various pieces of information from the VideoCore GPU on the Raspberry Pi. The linux flexible I/O tester tool is  easy to use and useful for understanding storage sub-system performance.

```
sudo apt install -y libraspberrypi-bin fio
```

## USB Flash Disk

The SD card on the Pi will normally show as /dev/mmcblk0. The USB drive will normally show as /dev/sda. The following could be data destructive so check the enumeration before proceeding.

```
sudo fdisk -l
```

![USB Device](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/usb_dev.png)


Then create primary partition on USB device

```
sudo sfdisk /dev/sda << EOF
;
EOF
```

![USB Partition](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/usb_dev.png)

Then format and label the partition then mount and set permissions for the parition

```
sudo mkfs.ext4 -L SSD /dev/sda1
sudo mkdir /mnt/ssd
sudo mount /dev/sda1 /mnt/ssd
echo "LABEL=SSD  /mnt/ssd  ext4  defaults 0 2" | sudo cat /etc/fstab -
sudo chmod 777 .
```

![USB Mount](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/usb_ext4.png)

## SD Card Performance

Linux FIO tool will be used to measure sequential write performance of a 4GB file:

```
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=4G --readwrite=write
```

![SD Card Read Performance](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/sd_reads.png)

The tool output shows sequential read rate:

1.  IOPS=10.1k, BW=39.5MiB/s
2.  IOPS=10.1k, BW=39.3MiB/s
3.  IOPS=10.1k, BW=39.4MiB/s
4.  IOPS=10.1k, BW=39.3MiB/s

Similarly, the tool will be used to measure sequential read performance of a 4GB file:

```
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=4G --readwrite=read
```

![SD Card Write Performance](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/sd_writes.png)

The tool output shows sequential write rate:

1.  IOPS=5429, BW=21.2MiB/s
2.  IOPS=5128, BW=20.0MiB/s
3.  IOPS=5136, BW=20.1MiB/s
4.  IOPS=5245, BW=20.5MiB/s

With the SD card the performance bottleneck is the reader which supports peak bandwidth 50MiB/s. To test this I has a lower spec SanDisk Ultra card so I repeated test and got near exact throughput to the SanDisk Extreme.

The tool output shows sequential read rate:

1.  IOPS=10.1k, BW=39.4MiB/s

The tool output shows sequential write rate:

1.  IOPS=5245, BW=20.5MiB/s

## USB Flash Performance

The USB flash drive should deliver improved performance, first check it can be seen by the system and note its device. 

Then repeated the same performance tests using fio on the SSD. Linux FIO tool will be used to measure sequential write/read performance of a 4GB file:

```
cd /mnt/ssd
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=4G --readwrite=write
```

![USB Write Performance](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/usb_writes.png)

The tool output shows sequential write rate:

1.  IOPS=14.5k, BW=56.6MiB/s
2.  IOPS=14.4k, BW=56.4MiB/s
3.  IOPS=14.4k, BW=56.2MiB/s
4.  IOPS=11.9k, BW=46.6MiB/s

```
cd /mnt/ssd
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=/mnt/ssd/test --bs=4k --iodepth=64 --size=4G --readwrite=read
```

![USB Read Performance](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/usb_reads.png)

The tool output shows sequential read rate:

1.  IOPS=33.3k, BW=130MiB/s
2.  IOPS=37.0k, BW=148MiB/s
3.  IOPS=42.6k, BW=166MiB/s
4.  IOPS=42.5k, BW=166MiB/s

So for a very small investment inpwd the USB Flash Drives we've increased sequential read potential by 4X and write potential by 3X.  The Pi 4 firmware doesn't presently offer option for USB boot so the SD cards are needed but hopefully soon the firmware will get updated and this will