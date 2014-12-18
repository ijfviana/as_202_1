## Introducción

* [Upstart](http://upstart.ubuntu.com/) está desarrollado por ubuntu y tiene como objetivos
 * Equivalente y es compatible con SysV
 * Gestiona mejor el *hardware* plug-ing

Note:
* Upstart does the same job as the SysV scripts, but Upstart is designed to
 * better handle today’s dynamically changing hot-plug hardware, which can be connected to and disconnected from a computer while it’s still running.
 * Upstart provides SysV compatibility features, so you should be familiar with the SysV methods described earlier



## Diferencias con SysV

* No existe el fichero `/etc/inittab`
* En el directorio `/etc/init` se encuentran los ficheros de arranque de los servicios
* Estos *scripts* los ejecuta `/lib/init/upstart-job`

```bash
$ ls
acpid.conf           cups.conf         module-init-tools.conf
alsa-restore.conf    dbus.conf         mountall.conf
alsa-store.conf      dmesg.conf        mountall-net.conf
anacron.conf         failsafe.conf     mountall-reboot.conf
```

Note:
* A system that uses nothing but Upstart and its native scripts replaces both /etc/inittab and the runlevel-specific SysV startup script directories with scripts in the /etc/init directory.



## Diferencias con SysV (II)

* Los `scripts` de arranque son de la forma siguiente

```bash
/etc/init$ vi cups.conf
# cups - CUPS Printing spooler and server

description     "CUPS printing spooler/server"
author          "Michael Sweet <msweet@apple.com>"

start on (filesystem
          and (started dbus or runlevel [2345]))
stop on runlevel [016]

respawn
respawn limit 3 12

pre-start script
    [ -x /usr/sbin/cupsd ]

    # load modules for parallel port support
    if [ -r /etc/default/cups ]; then
	. /etc/default/cups
    fi
    if [ "$LOAD_LP_MODULE" = "yes" -a -f /usr/lib/cups/backend/parallel \
	 -a -f /proc/modules -a -x /sbin/modprobe ]; then
	modprobe -q -b lp || true
	modprobe -q -b ppdev || true
	modprobe -q -b parport_pc || true
    fi

    mkdir -p /var/run/cups/certs
    if [ -x /lib/init/apparmor-profile-load ]; then
	/lib/init/apparmor-profile-load usr.sbin.cupsd
    fi
end script

exec /usr/sbin/cupsd -F

post-start script
    # wait until daemon is ready
    timeout=6
    while [ ! -e /var/run/cups/cups.sock ]; do
        sleep 0.5
	timeout=$((timeout-1))
	if [ "$timeout" -eq 0 ]; then
	    echo "cupsd failed to create /var/run/cups/cups.sock, skipping automatic printer configuration" >&2
	    exit 0
	fi
    done

    # coldplug USB printers
    if ! /lib/udev/udev-configure-printer enumerate 2>/dev/null; then
        if type udevadm > /dev/null 2>&1 && [ -x /lib/udev/udev-configure-printer ]; then
            for printer in `udevadm trigger --verbose --dry-run --subsystem-match=usb \
                    --attr-match=bInterfaceClass=07 --attr-match=bInterfaceSubClass=01 2>/dev/null || true; \
                            udevadm trigger --verbose --dry-run --subsystem-match=usb \
                    --sysname-match='lp[0-9]*' 2>/dev/null || true`; do
                /lib/udev/udev-configure-printer add "${printer#/sys}"
            done
        fi
    fi
end script
```

Note:
* To change the runlevels in which a particular service runs, you’ll have to edit its configuration file in a text editor.
Locate the script (typically /etc/init/name.conf, where name is the name of the service), and load it into a text
editor.



## Diferencias con SysV (III)

* Hay que ejecutar [initctl](http://linux.die.net/man/8/initctl) tras modificar cualquier script
```bash
ijfviana@vagalume:/etc$  initctl reload
```
* Upstart no está del todo extendido por lo que tiene un modo de compatibilidad con SysV ejecutando los scipts de SysV

```bash
:/etc$ ls -ld rc* init.d/
drwxr-xr-x 2 root root 4096 oct 20 22:52 init.d/
drwxr-xr-x 2 root root 4096 sep 19 10:23 rc0.d
drwxr-xr-x 2 root root 4096 jul  7 21:15 rc1.d
drwxr-xr-x 2 root root 4096 jul  7 21:15 rc2.d
drwxr-xr-x 2 root root 4096 jul  7 21:15 rc3.d
drwxr-xr-x 2 root root 4096 jul  7 21:15 rc4.d
drwxr-xr-x 2 root root 4096 jul  7 21:15 rc5.d
drwxr-xr-x 2 root root 4096 sep 19 10:23 rc6.d
-rwxr-xr-x 1 root root 3033 mar 25  2013 rc.local
drwxr-xr-x 2 root root 4096 sep 19 10:23 rcS.d
```

Note:
* After you make such a change, you can use the start or stop command to immediately start or stop the service,
* Before changing your runlevel, you should type initctl reload to have Upstart reread its configuration files.
* If you upgrade the package that provides the Upstart configuration script, you may need to reconfigure again.
* Because the SysV startup script system has been so common for so long, a large number of software packages include SysV startup scripts.
* In order to accommodate such packages, Upstart provides a compatibility mode: It runs SysV startup scripts in the usual locations (/etc/rc.d/rc?.d, /etc/init.d/rc?.d, /etc/rc?.d, or a similar location).
* Thus, if you install a package that doesn’t yet include an Upstart configuration script, it should still launch in the usual way. Furthermore, if you’ve installed utilities such as chkconfig, you should be able to use them to manage your SysV-based services just as you would on a SysV-based system.
* You may find, however, that chkconfig and other SysV-based tools no longer work for some services. As time goes on, this is likely to be true for more and more services, because the developers of distributions that favor Upstart may convert their packages’ startup scripts to use Upstart-native methods.
