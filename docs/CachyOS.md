# CachyOS Tweaks

## Chromium-Based Browsers HW Acceleration

### Brave Browser 

```bash
sudo pacman -S --needed mesa libva-mesa-driver libva-utils
```

with the following flags

```bash
--use-gl=angle 
--use-angle=vulkan 
--enable-features=Vulkan,VulkanFromANGLE,DefaultANGLEVulkan,AcceleratedVideoDecodeLinuxZeroCopyGL,AcceleratedVideoEncoder,VaapiIgnoreDriverChecks,UseMultiPlaneFormatForHardwareVideo,VaapiVideoEncoder,VaapiVideoDecode 
--ozone-platform=x11
```

### ProtonVPN

For some reason need to disable `Kwallet Secret Service` in order to use FIDO authentication.
