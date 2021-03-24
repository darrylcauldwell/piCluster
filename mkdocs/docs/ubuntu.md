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

## VCGENCMD

The vvcgencmd is a command line utility that can get various pieces of information from the VideoCore GPU on the Raspberry Pi. This isn't shipped with Ubuntu but as Raspberry Pi OS is open source we can build from source.

```
sudo apt install -y gcc-aarch64-linux-gnu cmake
git clone https://github.com/raspberrypi/userland.git
cd userland
./buildme --aarch64
```

Now we have the binary we can copy it and a libary it depends and make it available.

```
sudo mv build/bin/vcgencmd /usr/local/bin/vcgencmd
sudo mv build/lib/libvcos.so /lib/
sudo chown root:video /usr/local/bin/vcgencmd
sudo chmod 775 /usr/local/bin/vcgencmd
sudo usermod -aG video ubuntu
echo 'SUBSYSTEM=="vchiq", GROUP="video", MODE="0660"' | sudo tee /etc/udev/rules.d/99-input.rules
sudo udevadm control --reload-rules && sudo udevadm trigger
```

## SD Card Performance

The linux flexible I/O tester tool is  easy to use and useful for understanding storage sub-system performance.

```
sudo apt-get install -y fio
```

Linux FIO tool will be used to measure sequential write performance of a 4GB file:

```
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=4G --readwrite=write
```

![SD Card Read Performance](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/sd_reads.png)

Similarly, the tool will be used to measure sequential read performance of a 4GB file:

```
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=4G --readwrite=read
```

![SD Card Write Performance](https://raw.githubusercontent.com/darrylcauldwell/piCluster/main/_images/sd_writes.png)

The tool output indcates me sequential reads:
1.  IOPS=10.1k, BW=39.5MiB/s
2.  IOPS=10.1k, BW=39.3MiB/s
3.  IOPS=10.1k, BW=39.4MiB/s
4.  IOPS=10.1k, BW=39.3MiB/s

The tool output indcates me sequential writes:
1.  IOPS=5429, BW=21.2MiB/s
2.  IOPS=5128, BW=20.0MiB/s
3.  IOPS=5136, BW=20.1MiB/s
4.  IOPS=5245, BW=20.5MiB/s

With the SD card the performance bottleneck is the reader which supports peak bandwidth 50MiB/s. To test this I has a lower spec SanDisk Ultra card so I repeated test and got near exact throughput to the SanDisk Extreme.

The tool output indcates me sequential reads:
1.  IOPS=10.1k, BW=39.4MiB/s

The tool output indcates me sequential writes:
1.  IOPS=5245, BW=20.5MiB/s

## USB Flash Performance

The USB flash drive should deliver improved performance,  with it attached I created a parition and mount point.

```
# Check that the USB drive is seen by the system
lsusb

# Run fdisk(/sbin/fdisk) as root to list drives and partitions
# The USB external drive will normally show as /dev/sda if you only have the one drive currently plugged in
# The SD card on the Pi will normally show as /dev/mmcblk0
sudo fdisk -l

## Create primary partition on USB
fdisk /dev/sda

## Format the partition
mkfs -t ext4 is equivalent to mkfs.ext4
mkfs.ext4 -L SSD /dev/sda1

## Mount the partition
sudo mkdir /mnt/ssd
sudo mount /dev/sda1 /mnt/ssd

## Persist mounts
sudo vi /etc/fstab

LABEL=SSD  /mnt/ssd  ext4  defaults 0 2
```

Then repeated the same performance tests using fio on the SSD.

The tool output indcates me sequential reads:
1.  

The tool output indcates me sequential writes:
1.  