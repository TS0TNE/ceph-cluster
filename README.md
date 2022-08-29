#### The test was done on ceph with these parrameters

> - The ceph cluster consists of 8 Nodes.
> - One 7.68TB SAS SSD disk on each node
> - 2 OSD services are running on each disk. (16 OSDs in POOL)
> - CPU Frequency of nodes are 2.60GHZ and 2.70GHZ
> - Three mon services in total
> - Network consists of 10GB/ps interfaces in bonding. Bonding with 40GP/ps on each node.
> - There are 4 different POOLS on same disks, each POOL is run with different Plugin.
> - OMAP with SASSSD in table below means that OMAP is located on the POOL (with replica N 3) with same disks., OMAP with PCISSD means that OMAP is located on the POOL (with replica N 3) with different PCISSD disks.
> - Where there is not an OMAP information included that means OMAP is on SASSSD
> - Tests where done from virtual machine (Kernel 5.19) with disabled rbd cache


##### Sequencial read 1mb single thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=Seqread -bs=1m  -iodepth=1 -rw=read -size=10G -filename=/dev/sdX
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     | 529MB/s       |          | 504	     |		  |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     | 407MB/s       |          | 388	     |		  |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     | 431MB/s       |          | 410	     |		  |
| jerasure   | liber8tion     |        | 6     | 2     | 8     | 298MB/s       | 	  | 284	     |		  |
| replica 3  | 		      |        |       |       |       | 514MB/s       | 	  | 490	     |		  |

##### Sequencial read 4mb single thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=Seqread -bs=4m  -iodepth=1 -rw=read -size=10G -filename=/dev/sdX
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     | 930MB/s       |          | 221       |            |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     | 743MB/s       |          | 177       |            |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     | 795MB/s       |          | 189       |            |
| jerasure   | liber8tion     |        | 6     | 2     | 8     | 613MB/s       |          | 146       |            |
| replica 3  |                |        |       |       |       | 932MB/s       |          | 222       |            |


##### Sequencial write 1mb single thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=SeqWrite -bs=1m  -iodepth=1 -rw=write -size=10G -filename=/dev/sdX -allow_mounted_write=1
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     |       	       | 155MB/s  |           | 147        |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     |               | 154MB/s  |           | 146        |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     |               | 156MB/s  |           | 149        |
| jerasure   | liber8tion     |        | 6     | 2     | 8     |               | 155MB/s  |           | 147        |
| replica 3  |                |        |       |       |       |               | 153MB/s  |           | 145        |


##### Same, but with write back option enabled - big difference

| Plugin     | Technique      | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | 4     | 2     | 8     |       	      | 1534MB/s |           | 1463       |
| jerasure   | blaum_roth     | 4     | 2     | 7     |     	      | 1363MB/s |           | 1299       |
| jerasure   | liber8tion     | 6     | 2     | 8     |        	      |          |        |            |


##### Sequencial write 4mb single thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=SeqWrite -bs=4m  -iodepth=1 -rw=write -size=10G -filename=/dev/sdX -allow_mounted_write=1
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     |               | 161MB/s  |           | 38         |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     |               | 176MB/s  |           | 42         |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     |               | 197MB/s  |           | 47         |
| jerasure   | liber8tion     |        | 6     | 2     | 8     |               | 220MB/s  |           | 52         |
| replica 3  |                |        |       |       |       |               | 181MB/s  |           | 43         |

##### Sequencial read 4mb 64 thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=Seqread -bs=4m  -iodepth=64 -rw=read -size=10G -filename=/dev/sdX
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     | 2219MB/s      |          | 529       |            |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     | 1746MB/s      |          | 416       |            |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     | 2687MB/s      |          | 540       |            |
| jerasure   | liber8tion     |        | 6     | 2     | 8     | 3137MB/s      |          | 747       |            |
| replica 3  |                |        |       |       |       | 3070MB/s      |          | 731       |            |


##### Sequencial write 4mb 64 thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=SeqWrite -bs=4m  -iodepth=64 -rw=write -size=10G -filename=/dev/sdX -allow_mounted_write=1
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     |               | 1645MB/s |           | 392        |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     |               | 1702MB/s |           | 405        |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     |               | 2311MB/s |           | 551        |
| jerasure   | liber8tion     |        | 6     | 2     | 8     |               | 1976MB/s |           | 471        |
| replica 3  |                |        |       |       |       |               | 1793MB/s |           | 427        |


##### Random read 512K 1 thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=512KRandRead --bs=512k -iodepth=1 --rw=randread -size=10G -filename=/dev/sdX
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     | 208MB/s       |          | 396       |            |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     | 208MB/s       |          | 396       |            |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     | 211MB/s       |          | 401       |            |
| jerasure   | liber8tion     |        | 6     | 2     | 8     | 203MB/s       |          | 386       |            |
| replica 3  |                |        |       |       |       | 257MB/s       |          | 490       |            |


##### Random read 512K 64 thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=512KRandRead --bs=512k -iodepth=64 --rw=randread -size=10G -filename=/dev/sdX
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     | 2256MB/s      |          | 4303      |            |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     | 2720MB/s      |          | 5187      |            |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     | 3078MB/s      |          | 5869      |            |
| jerasure   | liber8tion     |        | 6     | 2     | 8     | 3130MB/s      |          | 5970      |            |
| replica 3  |                |        |       |       |       | 1916MB/s      |          | 3655      |            |



##### Random read 4K 1 thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=4KRandRead --bs=4k -iodepth=1 --rw=randread -size=1G -filename=/dev/sdX
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     | 5950kB/s      |          | 1452      |            |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     | 5881kB/s      |          | 1435      |            |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     | 4743kB/s      |          | 1157      |            |
| jerasure   | liber8tion     |        | 6     | 2     | 8     | 4232kB/s      |          | 1033      |            |
| replica 3  |                |        |       |       |       | 8029kB/s      |          | 1960      |            |



##### Random read 4K 128 thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=4KRandRead --bs=4k -iodepth=128 --rw=randread -size=10G -filename=/dev/sdX
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     | 247MB/s       |          | 60.3k     |            |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     | 249MB/s       |          | 60.8k     |            |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     | 136MB/s       |          | 33.3k     |            |
| jerasure   | liber8tion     |        | 6     | 2     | 8     | 109MB/s       |          | 26.5k     |            |
| replica 3  |                |        |       |       |       | 304MB/s       |          | 74.1k     |            |



##### Random read 8K 128 thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=4KRandRead --bs=8k -iodepth=128 --rw=randread -size=10G -filename=/dev/sdX
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     | 471MB/s       |          | 57.5k     |            |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     | 454MB/s       |          | 55.5k     |            |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     | 287MB/s       |          | 35.0k     |            |
| jerasure   | liber8tion     |        | 6     | 2     | 8     | 210MB/s       |          | 25.6k     |            |
| replica 3  |                |        |       |       |       | 568MB/s       |          | 69.4k     |            |


##### Random write 4K 128 thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=4KRandRead --bs=4k -iodepth=128 --rw=randwrite -size=2G -filename=/dev/sdX -allow_mounted_write=1
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     |       	       | 34.2MB/s |           | 8344       |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     |               | 31.2MB/s |           | 7616       |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     |               | 25.5MB/s |           | 6226       |
| jerasure   | liber8tion     |        | 6     | 2     | 8     |               | 25.3MB/s |           | 6169       |
| replica 3  |                |        |       |       |       |               | 102MB/s  |           | 24.0k      |


##### Random write 8K 128 thread
```bash
fio -ioengine=libaio -direct=1 -invalidate=1 -name=4KRandRead --bs=8k -iodepth=128 --rw=randwrite -size=2G -filename=/dev/sdX -allow_mounted_write=1
```

| Plugin     | Technique      | OMAP   | K     | M     | W     | Bw read       | Bw write | IOPS read | IOPS write |
| :---       | :---:          | :---:  | :---: | :---: | :---: | ------------: | :---:    | :---:     | :---:      |
| jerasure   | reed_sol_van   | SASSSD | 4     | 2     | 8     |               | 69.2MB/s |           | 8447       |
| jerasure   | reed_sol_van   | PCISSD | 4     | 2     | 8     |               | 71.6MB/s |           | 8744       |
| jerasure   | blaum_roth     |        | 4     | 2     | 7     |               | 51.7MB/s |           | 6315       |
| jerasure   | liber8tion     |        | 6     | 2     | 8     |               | 51.2MB/s |           | 6246       |
| replica 3  |                |        |       |       |       |               | 208MB/s  |           | 25.3k      |

