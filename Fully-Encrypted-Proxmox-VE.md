# Installing a fully encrypted Proxmox with SSH unlock

> [!NOTE]  
> All commands run as `root`

1. Install Debian Bookworm (12), select encrypted LVM
1. Copy your SSH key across and connect with SSH
1. When in the new installation, install packages:

    ```bash
    apt install -y sudo dropbear-initramfs systemd-boot
    usermod -a -G sudo <your_user>
    ```

1. Copy your SSH key to the `/etc/dropbear/initramfs/authorized_keys` file. 
1. Update the host keys so it doesn't trigger an SSH warning (this assumes that
   the keys in your user's `.ssh/authorized_keys` file are the ones you want
    to use):
    ```bash
    for I in ed25519 rsa ecdsa
    do
    /usr/lib/dropbear/dropbearconvert openssh dropbear "/etc/ssh/ssh_host_${I}_key" "/etc/dropbear/initramfs/dropbear_${I}_host_key"
    done

    sed -e 's/^ssh-/no-port-forwarding,no-agent-forwarding,no-x11-forwarding,command="\/bin\/cryptroot-unlock" &/' ~<your_user>/.ssh/authorized_keys > /etc/dropbear/initramfs/authorized_keys
    ```

1. Update the `/etc/dropbear/initramfs/dropbear.conf` and set the options for the `dropbear` daemon:

    ```conf
    DROPBEAR_OPTIONS="-s -j -k -I 60" 
    ```

1. Generate the `initramfs` and reboot:

    ```bash
    update-initramfs -u -k all
    systemctl reboot
    ```

1. Try to log in and enter your disk password. `ssh root@<server>`
1. Follow the instructions on the [Proxmox wiki to install](https://pve.proxmox.com/wiki/Install_Proxmox_VE_on_Debian_12_Bookworm). Pay attention to changing the `/etc/hosts` file!
1. Unmount the EFI boot partition, and let Proxmox handle it:

    ```bash
    ESP_DEV=$(findmnt /boot/efi --noheadings --output source)
    umount $ESP_DEV
    proxmox-boot-tool init $ESP_DEV grub
    update-initramfs -u -k all
    systemctl reboot
    ```

1. Add ZFS pool, set up network etc.
1. Install mail packages for gmail: `apt install -y libsasl2-modules mailutils`
1. Set an email for the root user and follow the guide here: https://www.naturalborncoder.com/2023/05/setting-up-email-notifications-in-proxmox-using-gmail/
1. I also like to set the management interface to the BMC unmanaged (`nmcli dev set enxd<tab> managed no`)
