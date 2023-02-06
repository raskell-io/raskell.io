---
title: "My OpenBSD journey: Getting it virtualized with libvirt (1)"
author: "Raffael"
date: "2019-01-11"
description: "My OpenBSD journey: Getting it virtualized with libvirt (1)"
tags:
- openbsd, virtualization
categories:
- foss
---

![openbsd-libvirt.png](openbsd-libvirt.png)

## Void Linux as my daily driver

Void Linux and OpenBSD are both Unix-like operating systems, but they have some differences. Here are a few similarities and differences between the two:

Similarities:

- Both are open-source operating systems.
- Both use a package manager for software management. Void Linux uses XBPS, while OpenBSD uses pkg_add.
- Both prioritize security and stability in their development and design.

Differences:

- License: Void Linux is licensed under the MIT License, while OpenBSD is licensed under the ISC License.
- Philosophy: OpenBSD prioritizes security and privacy, while Void Linux prioritizes simplicity and modularity.
- Package Management: Void Linux uses a binary package manager (XBPS), while OpenBSD uses a source-based package manager (pkg_add).
- Package Repository: Void Linux has a large and diverse repository, while OpenBSD has a smaller and more curated repository.
- Init System: Void Linux uses runit as its init system, while OpenBSD uses rc.

## Why OpenBSD

OpenBSD is a free and open-source operating system that focuses on security, standardization, and robustness. It is based on the Berkeley Software Distribution (BSD) Unix operating system and is developed by a global community of volunteers. OpenBSD aims to provide a secure platform for both personal and enterprise use by implementing strong security features, including access control mechanisms, encryption, and auditing.

Theo de Raadt is the founder and lead developer of OpenBSD. His main objective with OpenBSD is to create a secure operating system that is free from backdoors, vulnerabilities, and other security weaknesses. He is committed to auditing the source code of the operating system and third-party software included with it, to identify and remove any potential security risks. De Raadt is also dedicated to improving the overall quality of the codebase and ensuring compatibility with a wide range of hardware and software.

### Virtualization

libvirt is an open-source virtualization management library that provides a simple and unified API for managing virtualization technologies, including KVM, QEMU, Xen, and others. It aims to simplify the process of creating, managing, and migrating virtual machines, storage, and networks, and to make it easier for administrators to manage virtual environments.

To virtualize an operating system ISO like OpenBSD with libvirt, you need to follow these steps:

1. Install libvirt and the virtualization technology you want to use, such as KVM.
2. Download the OpenBSD ISO file and place it in a location accessible by libvirt.
3. Create a new virtual machine in libvirt with the OpenBSD ISO as the installation media. This can be done through the command line or using a graphical user interface such as virt-manager.
4. Configure the virtual machine, including the amount of memory, CPU, and disk space, to meet the requirements of OpenBSD.
5. Start the virtual machine and install OpenBSD as you would on a physical machine.
6. Once the installation is complete, you can configure the virtual network, storage, and other settings as required.
7. Finally, you can use the libvirt API or the command line to manage and control the virtual machine, including starting, stopping, migrating, and snapshotting.

Installed dependencies

```bash
❯ sudo xbps-install -S dbus qemu libvirt virt-manager virt-viewer tigervnc
```

```bash
❯ sudo usermod -aG libvirt zara
```

Got ISO

```bash
# cd /var/lib/libvirt/boot/
sudo wget https://mirror.ungleich.ch/pub/OpenBSD/7.2/amd64/install72.iso
--2023-01-12 20:48:15--  https://mirror.ungleich.ch/pub/OpenBSD/7.2/amd64/install72.iso
Resolving mirror.ungleich.ch (mirror.ungleich.ch)... 2a0a:e5c0:2:2:400:c8ff:fe68:bef3, 185.203.114.135
Connecting to mirror.ungleich.ch (mirror.ungleich.ch)|2a0a:e5c0:2:2:400:c8ff:fe68:bef3|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 583352320 (556M) [application/octet-stream]
Saving to: ‘install72.iso.1’

install72.iso.1                      100%[====================================================================>] 556.33M  7.11MB/s    in 77s     

2023-01-12 20:49:33 (7.19 MB/s) - ‘install72.iso.1’ saved [583352320/583352320]
sudo wget https://mirror.ungleich.ch/pub/OpenBSD/7.2/amd64/SHA256
--2023-01-12 20:47:38--  https://mirror.ungleich.ch/pub/OpenBSD/7.2/amd64/SHA256
Resolving mirror.ungleich.ch (mirror.ungleich.ch)... 2a0a:e5c0:2:2:400:c8ff:fe68:bef3, 185.203.114.135
Connecting to mirror.ungleich.ch (mirror.ungleich.ch)|2a0a:e5c0:2:2:400:c8ff:fe68:bef3|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1992 (1.9K) [application/octet-stream]
Saving to: ‘SHA256.1’

SHA256.1                             100%[====================================================================>]   1.95K  --.-KB/s    in 0s      

2023-01-12 20:47:39 (742 MB/s) - ‘SHA256.1’ saved [1992/1992]
# grep install63.iso SHA256 > /tmp/x
# sha256sum -c /tmp/x
# rm /tmp/x
```

Installed ISO

```bash
sudo virt-install \
      --name=openbsd \
      --virt-type=qemu \
      --memory=2048,maxmemory=4096 \
      --vcpus=2,maxvcpus=4 \
      --cpu host \
      --os-variant=openbsd7.0 \
      --cdrom=/var/lib/libvirt/boot/install72.iso \
      --network=bridge=virbr0,model=virtio \
      --graphics=vnc \
      --disk path=/var/lib/libvirt/images/openbsd.qcow2,size=40,bus=virtio,format=qcow2
```

Use VNC

**`virsh`** is a command-line interface tool for managing virtualization environments created with libvirt. It allows administrators to manage virtual machines, storage pools, and network interfaces, as well as other virtualization components, from the command line.

To establish a VNC connection with a running libvirt virtualized OpenBSD instance, you can use the following steps:

1. Start the virtual machine in libvirt: You can start the virtual machine using the virsh command **`virsh start <vm-name>`**, where **`<vm-name>`** is the name of the virtual machine you want to start.
2. Find the VNC display: Once the virtual machine is running, you can find the VNC display number for the virtual machine using the virsh command **`virsh vncdisplay <vm-name>`**.
3. Connect to the VNC display: You can connect to the VNC display using a VNC client, such as **`vncviewer`**, and specify the IP address of the host running the virtual machine and the VNC display number. For example, if the host's IP address is **`192.168.0.100`** and the VNC display number is **`:0`**, the command to connect would be **`vncviewer 192.168.0.100:0`**.
4. Authenticate to the VNC server: You may need to enter a password to authenticate to the VNC server. The password is set when the virtual machine is created in libvirt.

With these steps, you can establish a VNC connection with a running libvirt virtualized OpenBSD instance and interact with the virtual machine's graphical user interface.

```bash
$ virsh dumpxml openbsd | grep vnc
<graphics type='vnc' port='5900' autoport='yes' listen='127.0.0.1'>
```

- open vncviewer (provided by package tigernvc)

Install OpenBSD in VNC window

After first install batch

```bash
❯ sudo virsh --connect qemu:///system start openbsd
Domain 'openbsd' startedr
```
