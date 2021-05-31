# GRUB2 netboot

Use GRUB to network boot UEFI hardware.

## Requirements

For local development, install the dependencies for libvirt with UEFI.

* [UEFI with QEMU](https://fedoraproject.org/wiki/Using_UEFI_with_QEMU)

Ensure that you've gone through the [matchbox with docker](getting-started-docker.md) and [matchbox](matchbox.md) guides and understand the basics.

## Containers

Run `matchbox` according to [matchbox with Docker](getting-started-docker.md), but mount the [grub](../examples/groups/grub) group example. Then start the `poseidon/dnsmasq` Docker image, which bundles a `grub.efi`.

```sh
$ sudo docker run --rm --cap-add=NET_ADMIN quay.io/poseidon/dnsmasq -d -q --dhcp-range=172.18.0.43,172.18.0.99 --enable-tftp --tftp-root=/var/lib/tftpboot --dhcp-match=set:efi-bc,option:client-arch,7 --dhcp-boot=tag:efi-bc,grub.efi --dhcp-userclass=set:grub,GRUB2 --dhcp-boot=tag:grub,"(http;matchbox.foo:8080)/grub","172.18.0.2" --log-queries --log-dhcp --dhcp-option=3,172.18.0.1 --dhcp-userclass=set:ipxe,iPXE --dhcp-boot=tag:pxe,undionly.kpxe --dhcp-boot=tag:ipxe,http://matchbox.foo:8080/boot.ipxe --address=/matchbox.foo/172.18.0.2
```

## Client VM

Create UEFI VM nodes which have known hardware attributes.

```sh
$ sudo ./scripts/libvirt create-uefi
```

Create a VM to verify the machine network boots.

```sh
$ sudo virt-install --name uefi-test --boot=uefi,network --disk pool=default,size=4 --network=bridge=docker0,model=e1000 --memory=1024 --vcpus=1 --os-type=linux --noautoconsole
```
