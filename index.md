## Nvidia Driver
https://docs.01.org/clearlinux/latest/tutorials/nvidia.html

### Configure Nvidia Driver
```sh
# Copy the first cmd and enter the sudo password
sudo tee /etc/systemd/system/fix-nvidia-libGL-trigger.service > /dev/null <<'EOF'
[Unit]
Description=Fixes libGL symlinks for the NVIDIA proprietary driver
BindsTo=update-triggers.target

[Service]
Type=oneshot
ExecStart=/usr/bin/ln -sfv /opt/nvidia/lib/libGL.so.1 /usr/lib/libGL.so.1
ExecStart=/usr/bin/ln -sfv /opt/nvidia/lib32/libGL.so.1 /usr/lib32/libGL.so.1
EOF
# End of first cmd, don't copy this line

# Copy the remaining cmds
sudo systemctl daemon-reload
sudo systemctl add-wants update-triggers.target fix-nvidia-libGL-trigger.service

# Disable nouveau
sudo mkdir -p /etc/modprobe.d
printf "blacklist nouveau \noptions nouveau modeset=0 \n" | sudo tee /etc/modprobe.d/disable-nouveau.conf

# Configure dynamic linker
echo "include /etc/ld.so.conf.d/*.conf" |  sudo tee /etc/ld.so.conf
sudo mkdir -p /etc/ld.so.conf.d
printf "/opt/nvidia/lib \n/opt/nvidia/lib32 \n" | sudo tee /etc/ld.so.conf.d/nvidia.conf
sudo ldconfig

# Configure Xorg
sudo mkdir -p /etc/X11/xorg.conf.d
sudo tee /etc/X11/xorg.conf.d/nvidia-files-opt.conf > /dev/null <<'EOF'
Section "Files"
        ModulePath      "/usr/lib64/xorg/modules"
        ModulePath      "/opt/nvidia/lib64/xorg/modules"
EndSection
EOF
```

### Install DKMS
```sh
sudo swupd bundle-add kernel-native-dkms
sudo clr-boot-manager update
```

### Download Nvidia Driver
```sh
curl -O https://download.nvidia.com/XFree86/Linux-x86_64/455.28/NVIDIA-Linux-x86_64-455.28.run
```

### Install Nvidia Driver
> Save the Nvidia Driver install cmd below to a text file before reboot

Reboot
```sh
# Nvidia Driver install cmd
sudo sh NVIDIA-Linux-x86_64-455.28.run \
--utility-prefix=/opt/nvidia \
--opengl-prefix=/opt/nvidia \
--compat32-prefix=/opt/nvidia \
--compat32-libdir=lib32 \
--x-prefix=/opt/nvidia \
--x-module-path=/opt/nvidia/lib64/xorg/modules \
--x-library-path=/opt/nvidia/lib64 \
--x-sysconfig-path=/etc/X11/xorg.conf.d \
--documentation-prefix=/opt/nvidia \
--application-profile-path=/etc/nvidia/nvidia-application-profiles-rc.d \
--no-precompiled-interface \
--no-nvidia-modprobe \
--no-distro-scripts \
--force-libglx-indirect \
--glvnd-egl-config-path=/etc/glvnd/egl_vendor.d \
--egl-external-platform-config-path=/etc/egl/egl_external_platform.d  \
--dkms \
--silent
```

## CUDA
https://docs.01.org/clearlinux/latest/tutorials/nvidia-cuda.html

### Install GCC 8
```sh
sudo swupd bundle-add c-extras-gcc8
sudo mkdir -p /usr/local/cuda/bin
sudo ln -s /usr/bin/gcc-8 /usr/local/cuda/bin/gcc
sudo ln -s /usr/bin/g++-8 /usr/local/cuda/bin/g++
```

### Download CUDA
```sh
curl -O https://developer.download.nvidia.com/compute/cuda/11.1.0/local_installers/cuda_11.1.0_455.23.05_linux.run
```

### Install CUDA
```sh
sudo sh cuda_11.1.0_455.23.05_linux.run \
--toolkit \
--installpath=/opt/cuda \
--no-man-page \
--override \
--silent
```
