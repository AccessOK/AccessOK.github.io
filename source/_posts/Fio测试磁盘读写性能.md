---
title: Fio测试磁盘读写性能
date: 2023-12-25 10:27:28
description: Fio测试磁盘读写性能
type: "tags"
comments: true
categories:
- Linux
- Tools
- Test
tags:
- Linux
- Test
---
# 环境

内核：4.10

cpu：8

内存：8GB

硬盘：100GB

# 安装fio

fio 的全称是 flexible I/O tester，是常用的磁盘性能测试工具。fio 通过产生一系列的线程或进程来执行用户指定的特定类型 IO 操作。典型的用法是用户将需要模拟的 IO 负载写入到 job file 中。fio 支持多种 IO 引擎，通过 ioengine=io_uring，我们可以在 fio 中使用 io_uring 接口来测试磁盘性能。

```bash
yum install fio
```

# 测试

- 4k随机读

  ```bash
  fio -name=iouring_test -filename=/mnt/vdd/testfile -iodepth=128 -thread -rw=randread -ioengine=io_uring -sqthread_poll=1 -direct=1 -bs=4k -size=10G -numjobs=1 -runtime=120 -group_reporting
  ```

- 4k 随机写

  ```bash
  fio -name=iouring_test -filename=/mnt/vdd/testfile -iodepth=128 -thread -rw=randwrite -ioengine=io_uring -sqthread_poll=1 -direct=1 -bs=4k -size=10G -numjobs=1 -runtime=120 -group_reporting
  ```

- 随机读写

  ```bash
  fio -name=iouring_test -filename=/mnt/vdd/testfile -iodepth=128 -thread -rw=randrw -ioengine=io_uring -sqthread_poll=1 -direct=1 -bs=4k -size=10G -numjobs=1 -runtime=120 -group_reporting
  ```

- 4k 顺序读

  ```bash
  fio -name=iouring_test -filename=/mnt/vdd/testfile -iodepth=128 -thread -rw=read -ioengine=io_uring -sqthread_poll=1 -direct=1 -bs=4k -size=10G -numjobs=1 -runtime=120 -group_reporting
  ```

- 4k 顺序写

  ```bash
  fio -name=iouring_test -filename=/mnt/vdd/testfile -iodepth=128 -thread -rw=write -ioengine=io_uring -sqthread_poll=1 -direct=1 -bs=4k -size=10G -numjobs=1 -runtime=120 -group_reporting
  ```

- 4K顺序读写

  ```bash
  fio -name=iouring_test -filename=/mnt/vdd/testfile -iodepth=128 -thread -rw=rw -ioengine=io_uring -sqthread_poll=1 -direct=1 -bs=4k -size=10G -numjobs=1 -runtime=120 -group_reporting
  ```

说明：
filename=/dev/sdb1    测试文件名称，通常选择需要测试的盘的data目录。
direct=1         测试过程绕过机器自带的buffer。使测试结果更真实。
bs=16k          单次io的块文件大小为16k
bsrange=512-2048     同上，提定数据块的大小范围
size=5g  本次的测试文件大小为5g，以每次4k的io进行测试。
numjobs=30        本次的测试线程为30.
runtime=1000       测试时间为1000秒，如果不写则一直将5g文件分4k每次写完为止。
ioengine=psync      io引擎使用pync方式
rwmixwrite=30      在混合读写的模式下，写占30%
group_reporting     关于显示结果的，汇总每个进程的信息。

此外
lockmem=1g        只使用1g内存进行测试。
zero_buffers       用0初始化系统buffer。
nrfiles=8        每个进程生成文件的数量。
read 顺序读
write 顺序写
rw,readwrite 顺序混合读写
randwrite 随机写
randread 随机读
randrw 随机混合读写
