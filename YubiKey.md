---
layout: doc
title: YubiKey in Qubes
permalink: /doc/YubiKey/
---

Using YubiKey to Qubes authentication
=====================================

You can use YubiKey to enhance Qubes user authentication, for example to mitigate
risk of snooping the password. This can also slightly improve security when you have [USB keyboard](https://github.com/marmarek/qubes-app-linux-input-proxy).

There (at least) two possible configurations: using OTP mode and using challenge-reponse mode.

OTP mode
--------

This can be configured using
[app-linux-yubikey](https://github.com/adubois/qubes-app-linux-yubikey)
package. This package does not support sharing the same key slot with other
applications (it will deny further authentications if you try).

Contrary to instruction there, currently there is no binary packages in Qubes
repository and you need to compile it yourself. This can change in the future.

Challenge-reponse mode
----------------------

In this mode, your YubiKey will generate response based on secret key, and
random challenge (instead of counter). This means that it isn't possible to
generate response in advance even when someone get access to your YubiKey. This
makes reasonably safe to use the same YubiKey for other services (also in
challenge-response mode).

Same as in OTP case, you will need to setup your YubiKey, choose separate
password (other than your login password!) and apply the configuration.

To use this mode you need:

1. Configure your YubiKey for challenge-reponse HMAC-SHA1 mode, for example
   [following this
   tutorial](https://www.yubico.com/products/services-software/personalization-tools/challenge-response/)
2. Install `ykpers` package in template on which your USB VM is based.
3. Create `/usr/local/bin/yubikey-auth` script:

       #!/bin/sh

       key="$1"

       if [ -z "$key" ]; then
           echo "Usage: $0 <AESKEY> [<PASSWORD-HASH>]"
           exit 1
       fi

       # if password has given, verify it
       if [ -n "$2" ]; then
           # PAM appends \0 at the end
           hash=`head -c -1 | openssl dgst -sha1 -r | cut -f1 -d ' '`
           if [ "x$2" != "x$hash" ]; then
               exit 1
           fi
       fi

       challenge=`head -c64 /dev/urandom | xxd -c 64 -ps`
       # You may need to adjust slot number and USB VM name here
       response=`qvm-run -u root --nogui -p sys-usb "ykchalresp -2 -x $challenge"`

       correct_response=`echo $challenge | xxd -r -ps | openssl dgst -sha1 -macopt hexkey:$key -mac HMAC -r | cut -f1 -d ' '`

       test "x$correct_response" = "x$response"
       exit $?

4. Adjust USB VM name (`sys-usb` above), and possibly YubiKey slot number (`2`
   above), then make the script executable.
5. Edit `/etc/pam.d/xscreensaver` (or appropriate file if you are using other
   screen locker program). Add this line at the beginning:

       auth [success=done default=ignore] pam_exec.so expose_authtok quiet /usr/local/bin/yubikey-auth AESKEY PASSWORD-HASH

   Replace `AESKEY` with hex-encoded key configured in the first step, then
   replace `PASSWORD-HASH` with SHA1 hash for your YubiKey-linked password (other
   than your standard Qubes password). You can calculate it using this command:

       echo -n "PASSWORD" | openssl dgst -sha1

### Usage

When you want to unlock your screen, plug YubiKey into USB slot, then enter
password associated with YubiKey. If you configured so, YubiKey will request
confirmation by pressing button on it (it will blink).
When everything is ok, your screen will be unlocked.

In any case you can still use your login password, but do it in secure location
where no one can snoop your password.
