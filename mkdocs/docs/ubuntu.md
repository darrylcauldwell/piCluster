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
sudo apt install -y libraspberrypi-bin
```

# Storage Performance

The linux flexible I/O tester tool is  easy to use and useful for understanding storage sub-system performance.

```
sudo apt install -y fio
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

