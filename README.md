## Instructions for enabling S3 (deep sleep) state for ASUS ROG Zephyrus G14 laptop

After being fed-up with, constant crashes when sleeping using the default **s2idle** sleep state, I've used the following instructions to enable the classic S3 Suspend-to-RAM, so called deep sleep, state under Linux.

I think that in almost 100% of the crashes, the culprit was the **amdgpu** driver. Anyway, S3 now works and there are no more crashes sleeping.

Here are the instructions:

1. Install ACPI tools:

```bash
sudo pacman -S acpica
```

2. In a an empty directory run:

```bash
sudo acpidump -b
sudo iasl -e *.dat  -d dsdt.dat
```
A file called **dsdt.dsl** is created.

3. Edit the file as follows:
- Increase the last number in DefinitionBlock (e.g. from 0x01072009 to 0x01072010):
  - from: DefinitionBlock ("", "DSDT", 2, "ALASKA", "A M I ", 0x01072009)
  - to: DefinitionBlock ("", "DSDT", 2, "ALASKA", "A M I ", 0x01072010)
- Search for XS3 and change it to _S3
  - from: Name (XS3, Package (0x04)
  - to: Name (_S3, Package (0x04)
- Search for 'Name (SS3, Zero)', replace
  - from: Name (SS3, Zero)
  - to: Name (SS3, One)


4. Compile back the **dsdt.dsl** file and create cpio archive
```bash
iasl -tc dsdt.dsl # this creates dsdt.aml
mkdir -p kernel/firmware/acpi
cp dsdt.aml kernel/firmware/acpi
find kernel | cpio -H newc --create > acpi_override
sudo cp acpi_override /boot
```

5. Update **/etc/default/grub**
- Edit file **/etc/default/grub** and append "mem_sleep_default=deep" to **GRUB_CMDLINE_LINUX_DEFAULT** parameter.
- Add the line **GRUB_EARLY_INITRD_LINUX_CUSTOM="acpi_override"**
- Invoke your script to update grub, on Arch:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```


6. Reboot and check if S3 sleep is enabled
```bash
cat /sys/power/mem_sleep
```

Should output:

```bash
s2idle [deep]****
```