---
title: "Hugepages Configuration"
description: "In order to export flows to ntopng using PF_RING ZC, you must have Linux huge pages configured. This typically is done in three simple steps"
outline: [2, 3]
---
# Hugepages Configuration {#hugepages-configuration}

In order to export flows to ntopng using PF_RING ZC, you must have Linux huge pages configured. This typically is done in three simple steps

```console
echo 2048 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
mkdir /mnt/huge
mount -t hugetlbfs nodev /mnt/huge
```

You can check if the above commands succeeded by doing

```console
cat /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
```

and you should read 2048. If less it means that you don’t have enough available memory. If you have enough physical memory, probably this error means that you have too much cached memory that you can free with

```console
free && sync && echo 3 > /proc/sys/vm/drop_caches && free
```

After this command you can to setup hugepages again and it will hopefully work.
