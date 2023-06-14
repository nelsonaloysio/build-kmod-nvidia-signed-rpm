# build-kmod-nvidia-signed-rpm

Automatically signs the NVIDIA kernel modules and/or builds an RPM package to be layered using rpm-ostree, allowing to secure boot the OS (Fedora
[Silverblue](https://silverblue.fedoraproject.org/) and [Kinoite](https://kinoite.fedoraproject.org/) 36+) without making the deployment mutable.

> **Alternatively, check out [silverblue-akmods-keys@CheariX](https://github.com/CheariX/silverblue-akmods-keys)**
> **for a (better!) workaround to automatically sign any modules built by akmods, by layering just one single package.**

___

### Usage

```
usage: build-kmod-nvidia-signed-rpm [-h|--help]
                                    [-y|--assume-yes]
                                    [-n|--assume-no]
                                    [-u|--unsigned]
```

___

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

___

### Install Nvidia driver kernel modules

On Silverblue/Kinoite, first layer the required [Nvidia proprietary driver package](https://rpmfusion.org/Howto/NVIDIA#Determining_your_card_model) with:

```
rpm-ostree install akmod-nvidia
# rpm-ostree install akmod-nvidia-470xx # GeForce 600/700 series
# rpm-ostree install akmod-nvidia-390xx # GeForce 400/500 series
# rpm-ostree install akmod-nvidia-340xx # GeFore 8/9/200/300 series
```

Also install the additional packages below as needed:

* `xorg-x11-drv-nvidia-cuda`: CUDA driver libraries.
* `xorg-x11-drv-nvidia-power`: preserve memory allocation on suspend/resume.

Afterwards, reboot your machine.

#### Add kernel arguments

The following kernel arguments might be required to successfully load the drivers:

```
rpm-ostree kargs --append-if-missing=rd.driver.blacklist=nouveau \
                 --append-if-missing=modprobe.blacklist=nouveau \
                 --append-if-missing=nvidia-drm.modeset=1
```

___

### Create and enroll Machine Owner Key (MOK)

Create a new [Machine Owner Key](https://rpmfusion.org/Howto/Secure%20Boot) if required:

```
sudo bash -c '[ ! -e /etc/pki/akmods/certs/public_key.der ] && kmodgenca'
```

Enroll it afterwards (choose a password to be entered on next boot):

```
mokutil --import /etc/pki/akmods/certs/public_key.der'
```
___

### Generate RPM package with modules

Download or clone the repository to generate a new RPM file with the signed modules and layer the package:

```
sudo bash build-kmod-nvidia-signed-rpm --assume-yes
```

Append `--unsigned` if your modules are already signed (e.g., by using [silverblue-akmods-keys@CheariX](https://github.com/CheariX/silverblue-akmods-keys)).

___

### Update kernel/Nvidia driver

To later update the deployed kernel or Nvidia driver, remove the layered package when issuing the command:

```
rpm-ostree update --uninstall kmod-nvidia-signed
```

Reboot into your new deployment and execute the script again in order to layer the new Nvidia kernel modules:

```
sudo bash build-kmod-nvidia-signed-rpm --assume-yes
```

___

## Notes

* Many thanks to [@CheariX](https://github.com/chearix) for debugging the issue and coming up with a solution to sign compressed modules on Fedora 36+.

* For more information, please check the corresponding ticket: [fedora-silverblue#272](https://github.com/fedora-silverblue/issue-tracker/issues/272).
