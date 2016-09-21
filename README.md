# Bubblegum96-Snappy
1)Build gadget snap  

  git clone  https://github.com/uCRDev/Bubblegum96-Snappy  

  cd Bubblegum96-Snappy  
  cd gadget   
  snapcraft snap actions  //a gadget snap named bubblegum96-gadget_0.1.0_arm64.snap will be generated here   


2)Build kernel snap  
  cd kernel  
  git clone https://github.com/96boards-bubblegum/linux
  cd linux  
  git checkout bubblegum96-3.10  
  git am ../patchs/*  
  
  cd ..  
  cd snappy  
  snapcraft --target-arch arm64 snap  
  
  //for 96board, it only supports uInitrd. Do a trick here, replace uInitrd with initrd.img  
  cd snap  
  mkimage -A arm64 -T ramdisk -C none -n "snappy initrd" -a 1ffffc0 -d initrd.img uInitrd  
  cp uInitrd initrd.img  
  snapcraft --target-arch arm64 snap  

  
3) Build bootable image  

   sudo ./ubuntu-device-flash core 16 --channel edge --gadget ./gadget/bubblegum96-gadget_0.1.0_arm64.snap  --kernel ./kernel/bubblegum-kernel_3.10.0_arm64.snap --os ubuntu-core --verbose --developer-mode -o bubblegum96-SD.img   


4) Burn image to SD card  

  sudo dd if=bubblegum96-SD.img of=/dev/sdX bs=32M  

