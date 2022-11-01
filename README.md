# build-kmod-nvidia-signed-rpm

Automatically signs the NVIDIA kernel modules and builds an RPM package to be layered using rpm-ostree,
allowing to secure boot the OS (Fedora [Silverblue](https://silverblue.fedoraproject.org/) and
[Kinoite](https://kinoite.fedoraproject.org/) 36+) without requiring the deployment to be made mutable.

> **Alternatively, check out [silverblue-akmods-keys@CheariX](https://github.com/CheariX/silverblue-akmods-keys) for a (better!) workaround to sign any modules built by akmods.**

### Usage

```
usage: build-kmod-nvidia-signed-rpm [-h|--help]
                                    [-y|--assume-yes]
                                    [-n|--assume-no]
```

## Guide

### Before anything...

**This is a Work In Progress**, and as such it hasn't been thoroughly tested. Check if Fedora is updated first:

```
rpm-ostree refresh-md &&
rpm-ostree ugrade
```

If required, restart your OS in order to boot into the updated deployment. Afterwards, make sure you pin it with:

```
sudo ostree admin pin 0
```

That way, you have a savepoint to which you can rollback to in case anything goes south. **You have been warned!**

### Install & sign Nvidia driver kernel modules

1. On Silverblue/Kinoite, first layer the required [Nvidia proprietary driver package](https://rpmfusion.org/Howto/NVIDIA#Determining_your_card_model) with:

```
rpm-ostree install akmod-nvidia
# rpm-ostree install akmod-nvidia-470xx # GeForce 600/700 series
# rpm-ostree install akmod-nvidia-390xx # GeForce 400/500 series
# rpm-ostree install akmod-nvidia-340xx # GeFore 8/9/200/300 series
```

2. Reboot the OS, [create a new Machine Owner Key](https://rpmfusion.org/Howto/Secure%20Boot) (MOK) if needed and enroll it (choose a password to be entered on next boot):

```
sudo kmodgenca &&
sudo mokutil --import /etc/pki/akmods/certs/public_key.der
```

3. Download the script to generate a new RPM file with the signed modules and layer the newly created package:

```
sudo bash build-kmod-nvidia-signed-rpm --assume-yes
```

### Update kernel/nvidia driver

To later update the deployed kernel or Nvidia driver, remove the layered package and update the deployment:

```
rpm-ostree remove kmod-nvidia-signed &&
rpm-ostree update --install akmod-nvidia[-470xx,-390xx,-340xx]
```

Reboot into your new deployment and execute the script again in order to sign the new Nvidia kernel modules:

```
sudo bash build-kmod-nvidia-signed-rpm --assume-yes
```

## Notes

* This script is meant as a workaround to solve issues regarding immutable deployments and unsigned drivers.

* You might need the following kernel arguments added: `rpm-ostree kargs --append=rd.driver.blacklist=nouveau --append=modprobe.blacklist=nouveau --append=nvidia-drm.modeset=1`.

* Also install the additional packages `xorg-x11-drv-nvidia-cuda` (CUDA driver) and `xorg-x11-drv-nvidia-power` (preserve memory allocation on suspend/resume) if needed.

* Many thanks to [@CheariX](https://github.com/chearix) for debugging the issue and coming up with a solution to sign compressed modules on Fedora 36.

* For more information, please check the corresponding ticket: [fedora-silverblue#272](https://github.com/fedora-silverblue/issue-tracker/issues/272).
