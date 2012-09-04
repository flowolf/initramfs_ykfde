initramfs_ykfde
===============

an init for initramfs, that supports full disk encryption (fde) using the yubico yubikey AES challenge response

this script is meant as inspirational guideline on how one could realize 
full disk encryption using a yubikey and the challenge response feature
supported by yubikeys after version 2.2 
(http://www.yubico.com/challenge-response)


# Features of this script:
 * it is capable of using the yubikey challenge response to generate a password
 * it generates a new key on every bootup. (the challenge is changed)
 * it supports yubikey's slot 2
 * it supports two factor authentication (combining a user password and the
   challenge response from the yuibkey.


### you will need (in your initramfs and in your system)
*      ykchalresp (found on github, it needs to be available in your initramfs
          https://github.com/Yubico/yubikey-personalization)
*      uuidgen 
*      lvm
*      cryptsetup

## How this script works:
 your system boots into initramfs and runs 'init' (this script), your kernel
 has to deliver all dependencies, as this script is very basic.
 it further requests the challenge, stored on boot (on /boot/crypt-challenge)
 and combines the response with a usergiven password (that you have to provide)
 after unlocking your partitions it generates a new challenge, saves it to the
 beforementioned file and sets the new combination of user password and 
 response as new password on slot "3" of the luks header. the new password is
 temporaly written to /newroot/root/key which is assumed to be on your 
 encrypted device.

### init
to initialize your crypt device to work with this script do the following:
we will use slot 3 for the challenge response key, you can change that to any
available slot.
 $ cryptsetup luksKillSlot /dev/yourdev 3 (optional, if you had a key there)
 $ mount /boot
 $ uuidgen > /boot/crypt-challenge
 $ ykchalresp -2 "`cat /boot/crypt-challenge `" > ~/key 
we assume that ~/key is located on your encrypted partition, otherwise this
might be a security risk!
 $ vi ~/key (add a userpassword in front of the string)
you will type in this password in at every bootup, the script will combine 
your passphrase and the yubikey response.
 $ cryptsetup luksAddKey --key-slot 3 /dev/sda2 ~/key
 $ rm ~/key


