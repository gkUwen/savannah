## Install AnyDSL in Hifive Unreleased
Edited in May,2020

### Install OS in hardware
**1. Download the OS image in here<br>**
https://dl.fedoraproject.org/pub/alt/risc-v/repo/virt-builder-images/images/<br>
Download **Fedora-Developer-Rawhide-*.raw.xz** as well as the matching **Fedora-Developer-Rawhide-*-fw_payload-uboot-qemu-virt-smode.elf**<br>

**2. Install virt-builder<br>**
```
apt -y install libguestfs-tools
```
**3. Uncompress the image<br>**
```
unxz Fedora-Developer-Rawhide-*.raw.xz
```
**4. Optional: expand the disk image<br>**

```
truncate -r Fedora-Developer-Rawhide-*.raw expanded.raw
truncate -s 40G expanded.raw
virt-resize -v -x --expand /dev/sda4 Fedora-Developer-Rawhide-*.raw expanded.raw
virt-filesystems --long -h --all -a expanded.raw
virt-df -h -a expanded.raw 
```
**5. Install on the HiFive Unleashed SD card<br>**
```
sudo virt-builder \
    --source https://dl.fedoraproject.org/pub/alt/risc-v/repo/virt-builder-images/images/index \
    --no-check-signature \
    --arch riscv64 \
    --format raw \
    --hostname anydsl \
    --output /dev/sdXXX \
    --root-password password:riscv \
    fedora-rawhide-developer-*(your image name)
```
### Configure in AnyDSL
**1. change in config.sh<br>**
```
+  : ${LLVM_TARGETS:=Native;RISCV"}#: ${LLVM_TARGETS:="AArch64;AMDGPU;ARM;NVPTX;X86"}
-  #: ${BRANCH_RV:=release_80}
```

**2. change in setup.sh<br>**
Until now, llvm/release 10 still occur errors when compiling in RISCV. However, we can compile successfully by using code in current branch.
```
sdfsdf
```
**3. change some codes for compati<br>**
```
1. thorin/src/thorin/be/llvm/llvm.cpp line455
origin:
call = irbuilder_.CreateCall(irbuilder_.CreateExtractValue(closure, 0), args);
change
auto va = irbuilder_.CreateExtractValue(closure, 0); 
call = irbuilder_.CreateCall(llvm::cast<llvm::FunctionType>(va->getType()->getPointerElementType()),va,args,"",nullptr);

2. /home/uwen/anydsl/impala/src/intrinsicgen/main.cpp line38
origin
llvm_name = llvm::Intrinsic::getName(id)
change
llvm_name = llvm::Intrinsic::getName(id).str();
```
**4. compile and check whether AnyDSL can run successfully in in Hifive Unreleased<br>**
