# build-kmod-nvidia-signed-rpm

Automatically signs the NVIDIA kernel modules and builds an RPM package to be layered using rpm-ostree,
allowing to secure boot the OS (Fedora [Silverblue](https://silverblue.fedoraproject.org/) and
[Kinoite](https://kinoite.fedoraproject.org/) 36+) without requiring the deployment to be made mutable.

### Usage

```
usage: build-kmod-nvidia-signed-rpm [-h|--help]
                                    [-y|--assume-yes]
                                    [-n|--assume-no]
```

## Guide

1. On Silverblue/Kinoite, first layer the required [Nvidia proprietary driver package](https://rpmfusion.org/Howto/NVIDIA#Determining_your_card_model) with:

```
rpm-ostree install akmod-nvidia
# rpm-ostree install akmod-nvidia-470xx # GeForce 600/700 series
# rpm-ostree install akmod-nvidia-390xx # GeForce 400/500 series
# rpm-ostree install akmod-nvidia-340xx # GeFore 8/9/200/300 series
```

2. Reboot the OS, [create a new Machine Owner Key](https://rpmfusion.org/Howto/Secure%20Boot) (MOK) if needed and enroll it (choose a password to be entered on next boot):

```
sudo kmodgenca
sudo mokutil --import /etc/pki/akmods/certs/public_key.der
```

3. Generate a new RPM file with the signed modules and layer the newly created package:

```
sudo ./build-kmod-nvidia-signed-rpm --assume-yes
```

### Update kernel/nvidia driver

To later update the deployed kernel or Nvidia driver, remove the layered package and update the deployment:

```
rpm-ostree remove akmod-nvidia kmod-nvidia-signed
rpm-ostree update --install akmod-nvidia
```

Reboot into your new deployment and execute the script again in order to sign the new Nvidia kernel modules:

```
sudo ./build-kmod-nvidia-signed-rpm --assume-yes
```

## Notes

* This script is meant as a workaround to solve issues regarding immutable deployments and unsigned drivers.

* Many thanks to [@CheariX](https://github.com/chearix) for debugging the issue and coming up with a solution to sign compressed modules on Fedora 36.

* For more information, please check the corresponding ticket: [fedora-silverblue#272](https://github.com/fedora-silverblue/issue-tracker/issues/272).
