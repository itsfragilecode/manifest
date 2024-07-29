### PixelOS Build Guide for veux

The host system in this guide is Arch Linux.
Disk usage of the compiled tree is around 250 GB.

### Main Steps

#### Prerequisites

1. **System Preparation**
   Ensure your system is up-to-date and install the required packages:

   ```bash
   sudo pacman -Syu
   sudo pacman -S jdk8-openjdk git gnupg flex bison gperf base-devel zip curl zlib gcc-multilib lib32-glibc lib32-ncurses xorgproto libx11 lib32-zlib mesa libxml2 libxslt unzip repo git-lfs jdk11-openjdk noto-fonts ttf-dejavu libxcrypt-compat
   ```

2. **Configure Java Environment**
   Set Java 11 as the default environment:

   ```bash
   sudo archlinux-java set java-11-openjdk
   ```

#### Setting Up the Source Code

3. **Initialize the Repo**
   Set up the repo tool and initialize the PixelOS source tree:

   ```bash
   mkdir -p ~/android/pixelos
   cd ~/android/pixelos
   repo init -u https://github.com/PixelOS-AOSP/manifest.git -b fourteen
   ```

   Note: If you are using your fork, use the following command:

   ```bash
   repo init -u https://github.com/spezifisch/PixelOS-manifest -b xiaomi-veux
   ```

4. **Synchronize the Source Code**
   Fetch the source code. This step can be resumed if interrupted:

   ```bash
   repo sync
   ```

#### Configuring Git LFS

5. **Configure Git LFS**
   Set up Git Large File Storage and fetch missing large files:

   ```bash
   git lfs install
   git lfs pull
   ```

6. **Handle Git LFS Issues**
   If there are issues with partial repositories due to interruptions, navigate to the affected directories and pull the files again:

   ```bash
   cd vendor/gms/common/proprietary/system_ext/app/
   git lfs pull
   ```

#### Update Environment Variables

7. **Update Environment Variables**
   Update environment variables for the build process:

   ```bash
   vim ~/.zshrc
   ```

   Add the following lines:

   ```bash
   export JAVA_FONTS=/usr/share/fonts
   export JAVA_FONTS_PATH=/usr/share/fonts
   ```

   Reload the configuration:

   ```bash
   source ~/.zshrc
   ```

#### Editing Configuration Files

8. **Edit Configuration Files**
   Make necessary changes to configuration files using `vim`:

   ```bash
   vim device/xiaomi/veux/Android.bp
   vim device/xiaomi/veux/BoardConfig.mk
   vim device/xiaomi/veux/sepolicy/vendor/genfs_contexts
   vim build/make/core/Makefile
   vim vendor/gms/common/Android.bp
   vim vendor/gms/proprietary-files.txt
   vim kernel/xiaomi/sm6375/drivers/staging/qcacld-3.0/core/mac/src/pe/lim/lim_api.c
   ```

#### Build the ROM

9. **Build the ROM**
   Start the build process. Use `bacon` to create the flashable zip:

   ```bash
   mka bacon -j$(nproc --all)
   ```

10. **Check the Build**
    Once the build is complete, check the output and sign the build if necessary:

    ```bash
    ls -lh out/target/product/veux/PixelOS_veux-14.0-<date-time>.zip
    sha512sum out/target/product/veux/PixelOS_veux-14.0-<date-time>.zip
    ```

### Troubleshooting

#### Handling Genfscon Errors

If you encounter errors related to genfscon, comment out the problematic lines in the `sepolicy/vendor/genfs_contexts` file. You can find the changes made in the file `veux-fixes.git1763efa2.repo.diff`. Here is an example of how to comment out the problematic lines:

```diff
-genfscon sysfs /devices/platform/dummy_hcd.0/usb1/wakeup u:object_r:sysfs_wakeup:s0
-genfscon sysfs /devices/platform/soc/1628000.qcom,msm-eud/wakeup u:object_r:sysfs_wakeup:s0
+#genfscon sysfs /devices/platform/dummy_hcd.0/usb1/wakeup u:object_r:sysfs_wakeup:s0
+#genfscon sysfs /devices/platform/soc/1628000.qcom,msm-eud/wakeup u:object_r:sysfs_wakeup:s0
```

#### Suppressing Warnings for Packed Structures

If there are errors related to packed structures, adjust the compiler flags:

```bash
vim kernel/xiaomi/sm6375/Makefile
```

Append the following flag to `KBUILD_CFLAGS`:

```makefile
KBUILD_CFLAGS += -Wno-address-of-packed-member
```

#### Fix Missing Perl Library

Fix any missing Perl library issues, such as `libcrypt.so.1`:

```bash
sudo pacman -S libxcrypt-compat
```

**Resources**

- [PixelOS-AOSP Manifest](https://github.com/PixelOS-AOSP/manifest)
- [Arch Linux Java Environment Configuration](https://wiki.archlinux.org/title/Java)
- [Handling SELinux Contexts](https://source.android.com/security/selinux/validate)

