---
layout: page
title: Ubuntu
---

# Bell

```jsx
/etc/inputrc
set bell-style none
```

# Locale

```bash
apt install locales
sudo vim /etc/locale.gen
uncomment ru_RU.UTF-8
uncomment en_US.UTF-8
sudo locale-gen
#dpkg-reconfigure locale
#388
#.bash_aliases: export LC_ALL="ru_RU.UTF-8”
```

```bash
useradd -m NNN
passwd XXX
groupadd wheel
usermod -aG wheel NNN
apt install sudo

vim /etc/sudoers
%wheel ALL=(ALL) NOPASSWD:ALL
```

# Caps lock <-> esc

```bash
sudo apt install gnome-tweaks
gnome-tweaks
```

- Open tweak-tool and click on the *Keyboard & Mouse* section in the left menu bar.
- Click on the *Additional Layout Options* button on the left.
- Under *Caps Lock behavior* select *Swap Esc and CapsLock*.

# asan problems

## asan fix (turn off addr randomize)
<https://stackoverflow.com/questions/5194666/disable-randomization-of-memory-addresses/5194709#5194709>
<https://stackoverflow.com/questions/77894856/possible-bug-in-gcc-sanitizers>

```bash
echo 0 | sudo tee  /proc/sys/kernel/randomize_va_space
```

## asan gcc problems

<https://askubuntu.com/questions/1500017/ubuntu-22-04-default-gcc-version-does-not-match-version-that-built-latest-defaul>

```bash
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 1
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-12 1
```

# bpftrace fix

Error: `Could not resolve symbol: /proc/self/exe:BEGIN_trigger`

Fix: install debug symbols

<https://ubuntu.com/server/docs/debug-symbol-packages>

```bash
sudo apt install bpftrace-dbgsym
```

# docker

## docker compose 2

<https://proghunter.ru/articles/installing-docker-compose-v2-on-ubuntu>

```bash
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.27.1/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
```

## docker debian:jessie repos

```bash
  echo deb [trusted=yes] http://archive.debian.org/debian jessie main > sources.list
  echo deb [trusted=yes] http://archive.debian.org/debian-security jessie/updates main >> sources.list
  apt-get update
  apt-get -o Acquire::Check-Valid-Until=false update

```

# kvm
```bash
$ virt-install --virt-type kvm --name debian-11 --cdrom  ~/Downloads/debian-11.10.0-amd64-DVD-1.iso --os-variant debian11 --disk size=10 --memory 1024

$ virsh list --all
$ sudo virt-manager
```
