# Installing Docker in Termux
This repository contains instructions on how to install Docker in [Termux](https://termux.com/), a powerful terminal emulator for Android.
Use docker on your phone without Root. possibilities are endless now.

# Prerequisites
Before proceeding with the installation, make sure you have the following prerequisites:
- An Android device with Termux installed. You can download Termux from the [F-Droid](https://f-droid.org/packages/com.termux/) app store.
- Stable internet connection.

# Installation Steps
Follow the steps below to install Docker in Termux:
1. Open Termux on your Android device.

2. Update and upgrade the packages by running the following command:
```bash
pkg update -y && pkg upgrade -y
```

3. Install the necessary dependencies by running the following command:
```bash
pkg install qemu-utils qemu-common qemu-system-x86_64-headless wget -y
```

4. Create a seperate directory:
```bash
mkdir alpine && cd alpine
```

5. Download Alpine Linux 3.19 (virt optimized) ISO:
```bash
wget http://dl-cdn.alpinelinux.org/alpine/v3.19/releases/x86_64/alpine-virt-3.19.1-x86_64.iso
```

6. Create disk (note it won't actually take 5GB of space, more like 500-600MB):
```bash
qemu-img create -f qcow2 alpine.img 5G
```

7. Boot it up:
```bash
qemu-system-x86_64 -machine q35 -m 1024 -smp cpus=2 -cpu qemu64 -drive if=pflash,format=raw,read-only=on,file=$PREFIX/share/qemu/edk2-x86_64-code.fd -netdev user,id=n1,dns=8.8.8.8,hostfwd=tcp::2222-:22 -device virtio-net,netdev=n1 -cdrom alpine-virt-3.19.1-x86_64.iso -nographic alpine.img
```

8. Login with username ``root`` (no password)

9. Setup network (press Enter to use defaults):
```bash
localhost:~# setup-interfaces
 Available interfaces are: eth0.
 Enter '?' for help on bridges, bonding and vlans.
 Which one do you want to initialize? (or '?' or 'done') [eth0]
 Ip address for eth0? (or 'dhcp', 'none', '?') [dhcp]
 Do you want to do any manual network configuration? [no]
```
localhost:~# 
```bash
ifup eth0
```

10. Create an answerfile to speed up installation:
```bash
wget https://raw.githubusercontent.com/mabdulmoghni/termux-docker-no-root/main/answerfile
```
> **NOTE:** If you see any error like this: ``wget: bad address 'gist.githubusercontent.com'``. Then run this command
> ```bash
> echo -e "nameserver 192.168.1.1\nnameserver 1.1.1.1" > /etc/resolv.conf
> ```

11. Patch ``setup-disk`` to enable serial console output on boot:
```bash
sed -i -E 's/(local kernel_opts)=.*/\1="console=ttyS0"/' /sbin/setup-disk
```

12. Run setup to install to disk
```bash
setup-alpine -f answerfile
```

13. Once installation is complete, power off the VM (command ``poweroff``)

14. Boot again without cdrom:
```bash
qemu-system-x86_64 -machine q35 -m 1024 -smp cpus=2 -cpu qemu64 -drive if=pflash,format=raw,read-only=on,file=$PREFIX/share/qemu/edk2-x86_64-code.fd -netdev user,id=n1,dns=8.8.8.8,hostfwd=tcp::2222-:22,hostfwd=tcp::2375-:2375 -device virtio-net,netdev=n1 -nographic alpine.img
```

A - 
`nano run_qemu.sh`
In the text editor, write the following:
```bash
#!/bin/bash
qemu-system-x86_64 -machine q35 -m 1024 -smp cpus=2 -cpu qemu64 -drive if=pflash,format=raw,read-only=on,file=$PREFIX/share/qemu/edk2-x86_64-code.fd -netdev user,id=n1,dns=8.8.8.8,hostfwd=tcp::2222-:22,hostfwd=tcp::2375-:2375 -device virtio-net,netdev=n1 -nographic alpine.img
```
Save and close the file. In nano, you can do this by pressing Ctrl+X, then Y to confirm saving, and then Enter to confirm the filename.

B - chmod command: `chmod +x run_qemu.sh`

C - `./run_qemu.sh`

15. Update system and install docker:
```bash

echo "nameserver 8.8.8.8" > /etc/resolv.conf
echo "nameserver 8.8.4.4" >> /etc/resolv.conf

apk update && apk add docker
```

16. Start docker:
```bash
service docker start
```

17. Enable docker on boot:
```bash
rc-update add docker
```

18. Check docker install successfully or not:
```bash
docker run hello-world
```

19. Connect to Docker from the main box Termux - without login to alpine VM:  
Boot it up.
Login with username ``root`` (no password)
Edit file /etc/runlevels/default/docker
Update docker args line to be like below
```bash
command_args="${DOCKER_OPTS} -H tcp://0.0.0.0:2375 --iptables=false
```
Save file
```bash
service docker restart
```

# Some useful keys
- ``Ctrl+a x``: quit emulation
- ``Ctrl+a h``: toggle QEMU console

# Exit Alpine VM / Return back to Termux.
20. Install Docker
```bash
apt-get install docker
```

21. Boot Alpine VM on boot and connect to Docker daemon.
Update /data/data/com.termux/files/usr/etc/bash.bashrc file and Add below lines.  

```bash
echo "Update this configurations in /data/data/com.termux/files/usr/etc/bash.bashrc"
echo "starting qemu alpine vm and connecting to docker daemon"
echo "Using the ~/alpine/run_qemu.sh script"
echo "Remove it from bash.bashrc file if not needed anymore"
echo "It consumes memory and cpu :)"
echo "Starting vm in background"
echo "Wait few seconds for docker daemon to start"
cd ~/alpine && ./run_qemu.sh > /dev/null 2>&1  & 
cd ~
```

22. Now Run docker containers from termux
```bash
docker run hello-world
docker ps -a
```

## Congratulations 🕺, you just started docker on your phone. possibilities are endless now.

# Some useful keys
- ``Ctrl+a x``: quit emulation
- ``Ctrl+a h``: toggle QEMU console

# Usage
Now that Docker is installed in Termux, you can start using it to manage and run containers on your Android device. Refer to the official [Docker documentation](https://docs.docker.com/) for more information on how to use Docker.