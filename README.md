# virtio-mapped-permissions
virtio 9p "mapped" mode requires extended attributes to be set. Documentation doesn't exist.

Set up a shared dir in virt-manager using:
-  type=mount
-  driver=path
-  mode=mapped
-  write=immediate
-  source=/tank/fakesharename
-  target=9pshare

or from the cli with `/usr/bin/qemu-system-x86_64 \[...\] -fsdev local,security_model=mapped,writeout=immediate,id=fsdev-fs0,path=/tank/fakesharename -device virtio-9p-pci,id=fs0,fsdev=fsdev-fs0,mount_tag=9pshare,bus=pci.0,addr=0x7 `

`mapped` mode has the 'libvirt-qemu' user (primary group 'kvm') perform all the guest actions on the host dir, so make sure that user has perms on the host dir:

`chown -R libvirt-qemu:kvm /tank/fakesharename`

`mapped` mode then stores the guest's perceived file attributes in the host file's *extended attributes*. If those attributes are unset on a file (i.e. if you populated this directory from the host rather than from inside the guest) then the guest won't know what to do with the files. To fix this:

`find /tank/fakesharename -type d -exec sh -c " setfattr -n user.virtfs.gid -v 0sIQAAAA== \"\$@\"; setfattr -n user.virtfs.mode -v 0s7UEAAA== \"\$@\"; setfattr -n user.virtfs.uid -v 0sIQAAAA== \"\$@\"; " _ {} \;`

`find /tank/fakesharename -type f -exec sh -c " setfattr -n user.virtfs.gid -v 0sIQAAAA==  \"\$@\"; setfattr -n user.virtfs.mode -v 0spIEAAA== \"\$@\"; setfattr -n user.virtfs.uid -v 0sIQAAAA== \"\$@\"; " _ {} \;`

Those will set the guest perms to www-data, or root, or something. Doesn't matter; as long as they are set to something-god-please-anything, you'll be able to change them from inside the guest.

Honestly, I might consider this behavior a bug. Others would consider it a security feature. Go tell it on the mailing list.
