本文中以使用`qemu`安装`openEuler`为例.


# 准备对应的ISO镜像
```
wget -c https://repo.openeuler.org/openEuler-20.03-LTS/ISO/aarch64/openEuler-20.03-LTS-aarch64-dvd.iso
```


```
dd if=/dev/zero of=flash1.img bs=1M count=64
dd if=/dev/zero of=flash0.img bs=1M count=64
```


```
qemu-img create openEuler-image.img 20G
```

```
qemu-system-aarch64 \
    -machine virt \
    -m 1G,slots=2,maxmem=4G \
    -cpu max \
    -smp 4 \
    -netdev user,id=vnet,hostfwd=:127.0.0.1:0-:22 \
    -device virtio-net-pci,netdev=vnet \
    -drive file=openEuler-image.img,format=raw,if=none,id=drive0,cache=writeback \
    -device virtio-blk,drive=drive0,bootindex=0 \
    -drive file=openEuler-20.03-LTS-aarch64-dvd.iso,format=raw,if=none,id=drive1,cache=writeback \
    -device virtio-blk,drive=drive1,bootindex=1 \
    -drive file=flash0.img,format=raw,if=pflash \
    -drive file=flash1.img,format=raw,if=pflash
```

```
qemu-system-aarch64 \
    -machine virt,gic-version=max \
    -m 1G,slots=2,maxmem=4G \
    -cpu max \
    -smp 4 \
    -netdev user,id=vnet,hostfwd=:127.0.0.1:0-:22 \
    -device virtio-net-pci,netdev=vnet \
    -drive file=openEuler-image.img,format=raw,if=none,id=drive0,cache=writeback \
    -device virtio-blk,drive=drive0,bootindex=0 \
    -drive file=flash0.img,format=raw,if=pflash \
    -drive file=flash1.img,format=raw,if=pflash
```


# 从ISO镜像安装

先使用 `qemu-img create -f qcow2 openEulerDisk.img 20G` 创建一个`qcow2`格式的20G大小镜像空间.如果不指定格式,将默认使用`raw`格式.相对于`qcow2`来说, `raw`的速度更快, 但占用的空间大,`qcow2`格式空间大小,并不是在创建完成后就生成对应大小的镜像,而是动态分配空间大小.


从`iso`中引导镜像,加载完成后,将出现`openEuler`的安装引导界面,在界面的安装过程中选择需要安装的位置.
```
qemu-system-x86_64 -boot d -cdrom openEuler-20.03-LTS-x86_64-dvd.iso -m 1G,slots=2,maxmem=4G -hda openEulerDisk.img
```
