# Cuddly-Memory

![](res/drawing.svg)

## Nvidia container/docker problem with ubuntu21.10/pop21.10

Error:
```
Running hook #0:: error running hook: exit status 1, stdout: , stderr: nvidia-container-cli: container error: cgroup subsystem devices not found: unknown.
```

Ref: https://github.com/NVIDIA/nvidia-docker/issues/1447#issuecomment-989024636
```
# Add experimental repo to /etc/apt/sources.list.d/libnvidia-container.list
# deb https://nvidia.github.io/libnvidia-container/experimental/ubuntu18.04/$(ARCH)
sudo editor /etc/apt/sources.list.d/libnvidia-container.list
```

For pop:

Ref: https://gist.github.com/kuang-da/2796a792ced96deaf466fdfb7651aa2e

```
# Assume you already got /etc/apt/sources.list.d/libnvidia-container.list setup
# you also need to change some priority for apt
# create a file under /etc/apt/preferences.d or modify existing one add
Package: *
Pin: origin nvidia.github.io
Pin-Priority: 1002 # Greater priority than pop own repo
```

For both: upgrade the packages.

```
sudo apt update
sudo apt upgrade
```

## Btrfs cause disk to be completely stalled (even for SSD)

Probably related to btrfs quota, you can disable it, but will lose some features.

```
btrfs quota disable /
```

If you use `timeshift` you also need to disable `Settings > Users > Enable BTRFS qgroups` too, since `timeshift` seems to switch it back on later.
