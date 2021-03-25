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

## Summary

So for a very small investment in the USB Flash Drives we've increased sequential read potential by 4X and write potential by 3X over the SD card.  The Pi 4 firmware doesn't presently offer option for USB boot so the SD cards are needed but hopefully soon the firmware will get updated.