## Sharing file with 9pfs

```
fsdev_args=(
  # map local path $PWD
  -fsdev local,path=$PWD,id=workdir,security_model=none

  # create virtio device in vm (mount using value of mount_tag)
  -device virtio-9p-device,id=workdir,fsdev=workdir,mount_tag=workdir
)
```

To mount the filesystem:

```
mount -t 9p -o version=9p2000.L,trans=virtio "workdir" "/mnt"
```

## Booting with a virtio console

```
console_args=(
  # this creates a virtio-serial bus
  -device virtio-serial-device,id=virtio-serial0

  # mux=on allows us to use the virtio console for both console and monitor
  -chardev stdio,mux=on,id=console

  # this creates the virtconsole device (will be /dev/hvc0 in the vm)
  -device virtconsole,chardev=console,bus=virtio-serial0.0,id=port1,nr=1

  # disable regular serial port to avoid two devices using stdio
  # (we could also direct this somewhere else, e.g. file/socket/etc)
  -serial none

  # multiplex the monitor on stdio
  -mon chardev=console,mode=readline
)
```

## Connect a network device to existing libvirt bridge

```
net_args=(
  # create a virtio network device inside the vm
  -device virtio-net-device,id=net0,netdev=net0

  # attach it to a tap device on the host
  -netdev tap,id=net0,br=virbr0,helper=/usr/libexec/qemu-bridge-helper
)
```

## Command line example

```
qemu-system-x86_64 -m 1g -accel kvm -M microvm -nographic \
  -kernel arch/x86/boot/bzImage \
  -initrd initrd \
  -append 'console=hvc0' \
  "${fsdev_args[@]}" \
  "${console_args[@]}" \
  "${net_args[@]}"
```
