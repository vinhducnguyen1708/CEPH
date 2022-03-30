# Thiết lập QOS cho RBD

## Khi chưa thiết lập QoS cho rbd
- Kiểm thử `IOPS ghi ngẫu nhiên` trên một VM
```sh
fio --directory=/mnt/ --direct=1 --rw=randwrite --bs=4k --size=512M --numjobs=4 --runtime=60 --group_reporting --name=testfile --output=test_random_IOPS
```
- Kết quả tại file `test_random_IOPS`
```sh
testfile: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=1
...
fio-3.16
Starting 4 processes
testfile: Laying out IO file (1 file / 512MiB)

testfile: (groupid=0, jobs=4): err= 0: pid=3487: Wed Mar 30 11:23:53 2022
  write: IOPS=1633, BW=6533KiB/s (6689kB/s)(383MiB/60006msec); 0 zone resets
    clat (usec): min=453, max=47079, avg=2307.77, stdev=1283.98
     lat (usec): min=459, max=47089, avg=2329.92, stdev=1288.30
    clat percentiles (usec):
     |  1.00th=[ 1074],  5.00th=[ 1303], 10.00th=[ 1434], 20.00th=[ 1631],
     | 30.00th=[ 1778], 40.00th=[ 1909], 50.00th=[ 2057], 60.00th=[ 2212],
     | 70.00th=[ 2409], 80.00th=[ 2704], 90.00th=[ 3261], 95.00th=[ 4015],
     | 99.00th=[ 6915], 99.50th=[ 8979], 99.90th=[16581], 99.95th=[21103],
     | 99.99th=[31589]
   bw (  KiB/s): min= 4240, max= 9139, per=99.82%, avg=6520.07, stdev=223.00, samples=480
   iops        : min= 1059, max= 2284, avg=1629.72, stdev=55.78, samples=480
  lat (usec)   : 500=0.01%, 750=0.11%, 1000=0.49%
  lat (msec)   : 2=45.68%, 4=48.66%, 10=4.67%, 20=0.32%, 50=0.06%
  cpu          : usr=2.47%, sys=59.79%, ctx=98830, majf=0, minf=44
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,97999,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=6533KiB/s (6689kB/s), 6533KiB/s-6533KiB/s (6689kB/s-6689kB/s), io=383MiB (401MB), run=60006-60006msec

Disk stats (read/write):
  vda: ios=0/98055, merge=0/6627, ticks=0/118653, in_queue=16420, util=99.58%
```
*IPOS có giá trị: 1633*

## Thiết lập QoS cho rbd

*Tại node `ceph-mon`*

- Tìm rbd trên pool đã khởi tạo
```sh
rbd -p volumes_hdd ls
```
*Kết quả:*
```sh
[root@ceph-ms-mon ~]# rbd -p volumes_hdd ls
volume-f497b182-33ea-4ed5-8044-3e2f908c9a7b
```

- Thiết lập giới hạn iops cho rbd
```sh
rbd config image set volumes_hdd/volume-f497b182-33ea-4ed5-8044-3e2f908c9a7b rbd_qos_iops_limit 500
```
- Kiểm thử lại bằng lệnh ban đầu
```sh
fio --directory=/mnt/ --direct=1 --rw=randwrite --bs=4k --size=512M --numjobs=4 --runtime=60 --group_reporting --name=testfile --output=test_random_IOPS
```
*Kết quả:*
```sh
root@qos-ceph:~# cat test_random_IOPS
testfile: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=1
...
fio-3.16
Starting 4 processes

testfile: (groupid=0, jobs=4): err= 0: pid=3548: Wed Mar 30 11:31:18 2022
  write: IOPS=506, BW=2026KiB/s (2075kB/s)(119MiB/60018msec); 0 zone resets
    clat (usec): min=484, max=58708, avg=7699.60, stdev=13568.65
     lat (usec): min=489, max=58726, avg=7728.25, stdev=13570.12
    clat percentiles (usec):
     |  1.00th=[  725],  5.00th=[  889], 10.00th=[ 1012], 20.00th=[ 1188],
     | 30.00th=[ 1369], 40.00th=[ 1549], 50.00th=[ 1778], 60.00th=[ 2089],
     | 70.00th=[ 2573], 80.00th=[ 3785], 90.00th=[38536], 95.00th=[40633],
     | 99.00th=[42730], 99.50th=[43254], 99.90th=[45351], 99.95th=[47449],
     | 99.99th=[50070]
   bw (  KiB/s): min= 1471, max= 5832, per=99.91%, avg=2024.11, stdev=103.11, samples=480
   iops        : min=  367, max= 1458, avg=505.29, stdev=25.81, samples=480
  lat (usec)   : 500=0.01%, 750=1.33%, 1000=8.14%
  lat (msec)   : 2=48.32%, 4=22.93%, 10=3.42%, 20=0.18%, 50=15.66%
  lat (msec)   : 100=0.01%
  cpu          : usr=1.17%, sys=12.06%, ctx=32081, majf=0, minf=44
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,30406,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=2026KiB/s (2075kB/s), 2026KiB/s-2026KiB/s (2075kB/s-2075kB/s), io=119MiB (125MB), run=60018-60018msec

Disk stats (read/write):
  vda: ios=0/30404, merge=0/53, ticks=0/209620, in_queue=173420, util=99.87%
```