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

### Notes

* This script is meant as a workaround to solve issues regarding immutable deployments and unsigned drivers.

* Many thanks to [@CheariX](https://github.com/chearix) for debugging the issue and coming up with a solution to sign compressed modules on Fedora 36.

* For more information, please check the corresponding ticket: [fedora-silverblue#272](https://github.com/fedora-silverblue/issue-tracker/issues/272).