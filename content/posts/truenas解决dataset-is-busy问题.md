---
title: truenas解决dataset_is_busy问题
date: 2022-07-27 22:36:10
tags:
---

当在truenas中创建zvol并分配给虚拟机后，如果想要删除这个zvol就会出现错误

```
cannot destroy 'xxxxxxxx': dataset is busy
```

我从[Dataset is Busy · Issue #4442 · openzfs/zfs · GitHub](https://github.com/openzfs/zfs/issues/4442)找到了解决办法

```
# pvdisplay
  --- Physical volume ---
  PV Name               /dev/zd64
  VG Name               vg_name
  PV Size               20.00 GiB / not usable 4.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              5119
  Free PE               4863
  Allocated PE          256
  PV UUID               HhldEE-NiiF-BiQH-2j9u-esRu-WIfC-uLmyYC
# vgchange --activate n vg_name
# vgexport vg_name
# zfs destroy -r <name>
```
