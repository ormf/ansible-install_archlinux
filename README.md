<!--
SPDX-FileCopyrightText: 2023 David Runge <dvzrv@archlinux.org>
SPDX-License-Identifier: CC-BY-SA-4.0
-->

# archlinux.install_archlinux

An [Ansible](https://www.ansible.com/) role to install Arch Linux on a host, that is either booted into an Arch Linux install medium, or that is booted into a Linux distribution, that has both access to the system drives and can create chroots.
It supports both UEFI and legacy targets.

By default this role will create a GPT partition layout in which on UEFI systems an EFI partition and a root partition are created.
On legacy systems a GRUB partition, a boot partition and a root partition are created.
In either system the root partition is then encrypted as a LUKS container.

If two drives are present, both LUKS containers are opened and formatted as
RAID (either btrfs or MDADM). If only one drive is present, the LUKS container
is opened and formatted.

## Requirements

Either one or two system drives need to be dedicated for the installation (and must not be used by the running system).
The host needs to be accessible via SSH by a user that may elevate privileges to root (e.g. via `sudo`).

The following packages need to be installed (which is given when using an Arch Linux installation medium):

* btrfs-progs
* cryptsetup
* dosfstools
* gptfdisk
* python

## Role Variables

* `install_archlinux`: a dict for configuring the installation (unset by default)
  * `bootstrap`: a dict for configuring the bootstrap environment (if it is used)
    * `chroot_dir`: a string defining the directory which is created by extracting the bootstrap tarball (defaults to `root.x86_64`)
    * `mirrorlist_country`: a string defining for which country to download a mirrorlist (defaults to `all`). **NOTE**: Other than `all`, [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country codes are expected!
    * `mirrorlist_prefetch`: a boolean value indicating whether to prefetch a pacman mirrorlist (defaults to `true`)
    * `prefix_dir`: a string defining an absolute directory on the initial target host in which to download and to extract a bootstrap tarball (defaults to `/tmp`). **NOTE**: If the initial target host has less than 2GiB of RAM, consider using a non-tmpfs directory!
    * `sigkey`: a string defining the OpenPGP key fingerprint used to verify the signature for the bootstrap tarball (defaults to `3E80CA1A8B89F69CBA57D98A76A5EF9054449A5C`)
    * `version`: a string defining the version of the bootstrap tarball to download (defaults to `latest`)
  * `chroot_dir`: a string defining an absolute directory that is used on the initial target system for mounting relevant components to install a new system in a [chroot](https://man.archlinux.org/man/chroot.1) (defaults to `/mnt`)
  * `disk_password`: a string defining a password which is used for encrypting and decrypting the final target root filesystem using [cryptsetup](https://man.archlinux.org/man/core/cryptsetup/cryptsetup.8.en) (defaults to unset). **WARNING**: When using this role in an ansible-playbook and providing a `disk_password`, make sure to encrypt the data using [ansible-vault](https://man.archlinux.org/man/ansible-vault.1)!
  * `enabled_services`: a list of strings defining systemd services to enable on the final target (defaults to `['auditd', 'sshd', 'systemd-networkd', 'systemd-resolved', 'fstrim.timer']`)
  * `filesystems`: a dict describing the filesystems to setup
    * `boot`: a dict defining the settings for the boot filesystem
      `type`: a string defining the filesystem to use for the boot partition - one of `ext4` or `vfat` (defaults to `vfat`)
    * `root`: a dict defining the settings for the root filesystem
      * `devmapper_prefix`: a string defining the prefix for a device-mapper device (defaults to `root`) below `/dev/mapper`. A number (starting at `0`) is appended to the prefix (depending on the amount of disks), which makes up the decrypted device, if `install_archlinux.disk_password` is set (e.g. `/dev/mapper/root0`)
      * `mount_opts`: a dict defining mount options for the root filesystem (defaults to `['compress=zstd']`)
      * `raid_level`: a number defining the RAID level to use if two target devices are detected in `install_archlinux.system_disks` (defaults to `0`)
      * `raid_type`: a string defining the RAID type to use - one of `btrfs` or `mdadm` (defaults to `btrfs`)
      * `subvolumes`: a dict defining the subvolume names to create (the defaults are the currently only supported). **NOTE**: Subvolumes are created in a nested fashion (the default volume is *not* used for the root filesystem).
        * `root`: a string defining the subvolume name for the final target `/` directory (defaults to `@`)
        * `home`: a string defining the subvolume name used for the final target `/home` directory (defaults to `@home`)
      * `type`: a string defining the filesystem to use for the root partition - currently only `btrfs` is supported (defaults to `btrfs`)
  * `force_install_medium`: a boolean value indicating whether to force an installation medium environment (defaults to `false`). **NOTE**: This option is mainly for testing purposes.
  * `force_rescue_system`: a boolean value indicating whether to force a rescue system environment (defaults to `false`). **NOTE**: This option is mainly for testing purposes.
  * `hostname`: a string defining the final target's hostname (defaults to `archlinux`)
  * `no_log`: a boolean value indicating whether to show no logs (defaults to `true`). **WARNING**: If set to `false`, this will leak passwords into the ansible log output (e.g. LUKS encryption, user passwords, etc.)! Use at your own risk.
  * `packages`: a list of strings defining packages (or package groups) to be installed to the final target system (defaults to `['btrfs-progs', 'htop', 'linux', 'openssh', 'python', 'tree', 'vim']`). **NOTE**: The `base` package is always implied!
  * `system_disks`: a list of strings defining absolute paths to block devices on the initial target host to use for the installation (defaults to `[]`)

## Dependencies

The following roles are required to be available (as some of their tasks are
included):

* [archlinux.grub](https://gitlab.archlinux.org/dvzrv/ansible-grub)
* [archlinux.pacman](https://gitlab.archlinux.org/dvzrv/ansible-pacman) to configure pacman in a chroot
* [archlinux.mkinitcpio](https://gitlab.archlinux.org/dvzrv/ansible-mkinitcpio) to configure and generate initrd images in a chroot
* [archlinux.systemd_boot](https://gitlab.archlinux.org/dvzrv/ansible-systemd_boot)
* [archlinux.systemd_networkd](https://gitlab.archlinux.org/dvzrv/ansible-systemd_networkd)
* [archlinux.systemd_resolved](https://gitlab.archlinux.org/dvzrv/ansible-systemd_resolved)
* [archlinux.systemd_timesyncd](https://gitlab.archlinux.org/dvzrv/ansible-systemd_timesyncd)
* [archlinux.users](https://gitlab.archlinux.org/dvzrv/ansible-users) to configure users in a chroot

## Example Playbook

Install Arch Linux on hosts using disk encryption on system disk `/dev/sda`.

```yaml
- name: Install Arch Linux on a UEFI target host using a single encrypted disk
  hosts: servers
  roles:
    - role: archlinux.install_arch
  vars:
    install_archlinux:
      disk_password: "change.me.i.am.plaintext"
      hostname: newhost
      system_disks:
        - /dev/sda
    pacman:
      mirrorlist_country: DE
      mirrorlist_prefetch: true
    reflector:
      country: Germany
    systemd_boot:
      auto_update: true
      esp_path: /efi
      timeout: 3
    systemd_networkd:
      apply: true
      networks:
        ethernet:
          - Match:
              - Name: eth*
              - Name: en*
          - Network:
              - DHCP: "yes"
    systemd_resolved:
      dns:
        - 8.8.8.8
    systemd_timesyncd:
      timezone: Europe/Berlin
    users:
      - name: testuser
        authorized_keys:
          - fake_foo
        comment: a test user
        git_repos: []
        groups:
          - wheel
        password: hunter2
        state: present
        shell: /usr/bin/bash
```

```yaml
- name: Install Arch Linux on a legacy target host using a single encrypted disk
  hosts: servers
  roles:
    - role: archlinux.install_arch
  vars:
    grub:
      disable_submenu: true
      enable_cryptodisk: true
      timeout: 3
    install_archlinux:
      disk_password: "change.me.i.am.plaintext"
      hostname: newhost
      system_disks:
        - /dev/sda
    pacman:
      mirrorlist_country: DE
      mirrorlist_prefetch: true
    reflector:
      country: Germany
    systemd_networkd:
      apply: true
      networks:
        ethernet:
          - Match:
              - Name: eth*
              - Name: en*
          - Network:
              - DHCP: "yes"
    systemd_resolved:
      dns:
        - 8.8.8.8
    systemd_timesyncd:
      timezone: Europe/Berlin
    users:
      - name: testuser
        authorized_keys:
          - fake_foo
        comment: a test user
        git_repos: []
        groups:
          - wheel
        password: hunter2
        state: present
        shell: /usr/bin/bash
```

## License

GPL-3.0-or-later

## Author Information

David Runge <dvzrv@archlinux.org>
