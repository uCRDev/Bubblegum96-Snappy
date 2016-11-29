# Bubblegum96-Snappy

## Requirements
Make sure your build environment is based on `Ubuntu 16.04`. You need to install snappy tools.<br />
```bash
sudo apt-get update
sudo apt-get install -y build-essential u-boot-tools debootstrap gcc-4.9-aarch64-linux-gnu device-tree-compiler
sudo apt-get install ubuntu-snappy snapcraft snapd
```
Notes:Bubblegum96 kernel can't be cross complied by gcc 5+ aarch64-linux-gnu, so you have to make soft link to gcc 4.9 related toolchain.

## Build gadget snap
```bash
git clone https://github.com/uCRDev/Bubblegum96-Snappy
cd Bubblegum96-Snappy
cd gadget
snapcraft snap gadget
```
a gadget snap named bubblegum96-gadget_16.04-1.1_arm64.snap will be generated here

## Build kernel snap
```bash
cd ~/Bubblegum96-Snappy/kernel
git clone https://github.com/96boards-bubblegum/linux
cd linux
git checkout bubblegum96-3.10
git am ../patchs/*

cd ../snappy
snapcraft --target-arch arm64 snap
#for 96board, it only supports uInitrd. Do a trick here, replace uInitrd with initrd.img
cd prime
mkimage -A arm64 -T ramdisk -C none -n "snappy initrd" -a 1ffffc0 -d initrd.img uInitrd
cp uInitrd initrd.img
cd ..
snapcraft --target-arch arm64 snap
```

## Prepare model assertion

Before we can build image we need to prepare model assertion and sign it. <br />
First prepare json file describing model assertion: <br />
Example of model for Bubblegum96, ~/Bubblegum96-Snappy/image/TEMPLATE-model.json <br />
```json
{
 "type": "model",
 "authority-id": "<your accout id>",
 "brand-id": "<your accout id>",
 "series": "16",
 "model": "bubblegum96",
 "architecture": "arm64",
 "gadget": "bubblegum96-gadget",
 "kernel": "bubblegum96-kernel",
 "timestamp": "2016-11-22T12:00:00+00:00"
}
```
You can find your account id here:[https://myapps.developer.ubuntu.com/dev/account/](https://myapps.developer.ubuntu.com/dev/account/)

### Sign model:
Before signing model, first make sure that you have valid key which has been registered with your accout. <br />
If you do not have key, create new one: <br />
```bash
snap create-key
```
Make sure you register key with your account: <br />
```bash
snapcraft register-key
```
To check keys on your machine run <br />
```bash
snap keys
```

Sign model assertion:
```bash
cd ~/Bubblegum96-Snappy/image
cat bubblegum96-model.json | snap sign -k default &> bubblegum96.model
```

## Build bootable image
We will build image with ubuntu-image, install it with:
```bash
sudo snap install --stable --devmode ubuntu-image
```
Build image
```bash
cd ~/Bubblegum96-Snappy/image
cp ../gadget/bubblegum96-gadget_16.04-1.1_arm64.snap .
cp ../kernel/snappy/bubblegum96-kernel_3.10.0_arm64.snap .
sudo /snap/bin/ubuntu-image --channel stable --image-size 2G --extra-snaps bubblegum96-gadget_16.04-1.1_arm64.snap --extra-snaps bubblegum96-kernel_3.10.0_arm64.snap -o bubblegum96.img bubblegum96.model
```

## Burn image to SD card
```bash
sudo dd if=bubblegum96.img of=/dev/sdX bs=32M
sync
```
