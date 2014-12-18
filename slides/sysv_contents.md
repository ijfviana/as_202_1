## Introducción
* Esquema tradicional de arranque de servicios mediante *scripts*
* El proceso [`init`](http://es.wikipedia.org/wiki/Init) lanza una serie de *scripts* dependiendo del nivel de ejecución
* Los servicios a lanzar por nivel se pueden especificar
  * En el fichero [`/etc/inittab`](http://unixhelp.ed.ac.uk/CGI/man-cgi?inittab+5)
  * Añadiendo *scripts* de arranque al nivel deseado

Note:
In the past, most Linux distributions have used System V (SysV) startup scripts, which are named after the System V
version of Unix on which they originated. In the Linux implementation of SysV startup scripts, the kernel launches a
process called init, which reads its configuration file and, in following its instructions, launches a series of scripts
that can vary from one runlevel to another. As described later, in “Configuring Upstart,” a competing startup
system is becoming common on Linux.



## inittab (I)

```bash
$ vi /etc/initab

# Level to run in
id:2:initdefault:

# Boot-time system configuration/initialization script.
si::sysinit:/etc/rc.sysinit

# What to do in single-user mode.
~:S:wait:/sbin/sulogin

# /etc/init.d executes the S and K scripts upon change of runlevel.
#
# Runlevel 0 is halt.
# Runlevel 1 is single-user.
# Runlevels 2-5 are multi-user.
# Runlevel 6 is reboot.

l0:0:wait:/etc/rc 0
l1:1:wait:/etc/rc 1
l2:2:wait:/etc/rc 2
l3:3:wait:/etc/rc 3
l4:4:wait:/etc/rc 4
l5:5:wait:/etc/rc 5
l6:6:wait:/etc/rc 6

# What to do at the "3 finger salute".
ca::ctrlaltdel:/sbin/shutdown -t3 -r now

# Runlevel 2,3: getty on virtual consoles
# Runlevel   3: mgetty on terminal (ttyS0) and modem (ttyS1)
1:23:respawn:/sbin/mingetty tty1
2:23:respawn:/sbin/mingetty tty2
3:23:respawn:/sbin/mingetty tty3
4:23:respawn:/sbin/mingetty tty4
S0:3:respawn:/sbin/agetty ttyS0 9600 vt100-nav
S1:3:respawn:/sbin/mgetty -x0 -D ttyS1
```

Note:
* A typical /etc/inittab file contains many entries, and except for a couple of special cases, inspecting or changing the contents of this file is best left to experts.
* Once all the entries in /etc/inittab for your runlevel are executed, your boot process is complete, and you can log in.



## inittab (II)

* Cada entrada del fichero `inittab` tiene la siguiente sintaxis
```
id:runlevels:action:process
```

<div class="table-responsive">
  <table class="table table-hover table-condensed table-bordered">
<thead>
<tr>
<th width="300em">Opt</th>
<th>Descripción</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>id</code></td>
<td>Identificador único de hasta 4 caracteres</td>
</tr>
<tr>
<td><code>runlevels</code></td>
<td>Lista de niveles en los que se ejecutará</td>
</tr>
<tr>
<td><code>action</code></td>
<td>Cómo el proceso init trata la acción (wait, respawn)</td>
</tr>
<tr>
<td><code>process</code></td>
<td>Qué tiene que ejecutar <code>init</code></td>
</tr>
</tbody>
</table>
</div>

Note:
* Entries in /etc/inittab follow a simple format. Each line consists of four colon-delimited fields: id:runlevels:action:process
* Each of these fields has a specific meaning
* id: sequence of one to four characters that identifies the entry’s function. runlevels This field consists of a list of runlevels for which this entry applies. For instance, 345 means the entry is applicable to runlevels 3, 4, and 5.
* action: tell init how to treat the process. For instance, wait tells init to start the process once when entering a runlevel and to wait for the process’s termination, and respawn tells init to restart the process whenever it terminates (which is great for login processes). Several other actions are available; consult the man page for inittab for details.
* process: the process to run for this entry, including any options and arguments that are required.

If you alter the /etc/inittab file, the changes won’t take effect until you reboot the computer or type a command
such as telinit Q to have it reread this file and implement its changes. Thus, when making changes, you should
keep them simple or test their effects, lest problems occur later, after you’ve forgotten about changes to this file.



## SysV Startup Scripts (I)

* En *script* `/etc/init.d/rc` o `/etc/rc.d/rc` es el encargado de ejecutar de ejecutar una serie de *scripts* dependiendo del nivel de arranque
* El conjunto de *scripts* disponible para su ejecución suele encontrarse en `/etc/init.d`
```bash
/etc/init$ ls /etc/init.d/
acpid          killprocs         saned
acpi-support   lightdm           sendsigs
alsa-restore   lm-sensors        setvtrgb
alsa-store     modemmanager      single
anacron        module-init-tools skeleton
apache2        mysql             smartd
```

Note:
The /etc/init.d/rc or /etc/rc.d/rc script performs the crucial task of running all the scripts associated with the runlevel.



## SysV Startup Scripts (II)

Ejemplo de fichero de arraque SysV

```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides:          apache2
# Required-Start:    $local_fs $remote_fs $network $syslog $named
# Required-Stop:     $local_fs $remote_fs $network $syslog $named
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# X-Interactive:     true
# Short-Description: Start/stop apache2 web server
### END INIT INFO

set -e

SCRIPTNAME="${0##*/}"
SCRIPTNAME="${SCRIPTNAME##[KS][0-9][0-9]}"
if [ -n "$APACHE_CONFDIR" ] ; then
	if [ "${APACHE_CONFDIR##/etc/apache2-}" != "${APACHE_CONFDIR}" ] ; then
		DIR_SUFFIX="${APACHE_CONFDIR##/etc/apache2-}"
	else
		DIR_SUFFIX=
	fi
elif [ "${SCRIPTNAME##apache2-}" != "$SCRIPTNAME" ] ; then
	DIR_SUFFIX="-${SCRIPTNAME##apache2-}"
	APACHE_CONFDIR=/etc/apache2$DIR_SUFFIX
else
	DIR_SUFFIX=
	APACHE_CONFDIR=/etc/apache2
fi
if [ -z "$APACHE_ENVVARS" ] ; then
	APACHE_ENVVARS=$APACHE_CONFDIR/envvars
fi
export APACHE_CONFDIR APACHE_ENVVARS

ENV="env -i LANG=C PATH=/usr/local/bin:/usr/bin:/bin"
if [ "$APACHE_CONFDIR" != /etc/apache2 ] ; then
	ENV="$ENV APACHE_CONFDIR=$APACHE_CONFDIR"
fi
if [ "$APACHE_ENVVARS" != "$APACHE_CONFDIR/envvars" ] ; then
	ENV="$ENV APACHE_ENVVARS=$APACHE_ENVVARS"
fi

#edit /etc/default/apache2 to change this.
HTCACHECLEAN_RUN=auto
HTCACHECLEAN_MODE=daemon
HTCACHECLEAN_SIZE=300M
HTCACHECLEAN_DAEMON_INTERVAL=120
HTCACHECLEAN_PATH=/var/cache/apache2$DIR_SUFFIX/mod_disk_cache
HTCACHECLEAN_OPTIONS=""

APACHE_HTTPD=$(. $APACHE_ENVVARS && echo $APACHE_HTTPD)
if [ -z "$APACHE_HTTPD" ] ; then
	APACHE_HTTPD=/usr/sbin/apache2
fi
if [ ! -x $APACHE_HTTPD ] ; then
	echo "No apache MPM package installed"
	exit 0
fi

. /lib/lsb/init-functions

test -f /etc/default/rcS && . /etc/default/rcS

if [ -f /etc/default/apache2$DIR_SUFFIX ] ; then
	. /etc/default/apache2$DIR_SUFFIX
elif [ -f /etc/default/apache2 ] ; then
	. /etc/default/apache2
fi

APACHE2CTL="$ENV /usr/sbin/apache2ctl"
HTCACHECLEAN="$ENV /usr/sbin/htcacheclean"

PIDFILE=$(. $APACHE_ENVVARS && echo $APACHE_PID_FILE)
if [ -z "$PIDFILE" ] ; then
	echo ERROR: APACHE_PID_FILE needs to be defined in $APACHE_ENVVARS >&2
	exit 2
fi

check_htcacheclean() {
	[ "$HTCACHECLEAN_MODE" = "daemon" ] || return 1

	[ "$HTCACHECLEAN_RUN"  = "yes"    ] && return 0

	MODSDIR=$(. $APACHE_ENVVARS && echo $APACHE_MODS_ENABLED)
	[ "$HTCACHECLEAN_RUN"  = "auto" \
	  -a -e ${MODSDIR:-$APACHE_CONFDIR/mods-enabled}/disk_cache.load ] && \
		return 0

	return 1
}

start_htcacheclean() {
	if [ ! -d "$HTCACHECLEAN_PATH" ] ; then
		echo "... directory $HTCACHECLEAN_PATH does not exist!" >&2
		return 1
	fi
	$HTCACHECLEAN $HTCACHECLEAN_OPTIONS -d$HTCACHECLEAN_DAEMON_INTERVAL \
			-i -p$HTCACHECLEAN_PATH -l$HTCACHECLEAN_SIZE
}

stop_htcacheclean() {
	pkill -P 1 -f "htcacheclean.* -p$HTCACHECLEAN_PATH " 2> /dev/null || echo ...not running
}

pidof_apache() {
	# if there is actually an apache2 process whose pid is in PIDFILE,
	# print it and return 0.
	if [ -e "$PIDFILE" ]; then
		if pidof apache2 | tr ' ' '\n' | grep -w $(cat $PIDFILE); then
			return 0
		fi
	fi
	return 1
}

apache_stop() {
	if $APACHE2CTL configtest > /dev/null 2>&1; then
		# if the config is ok than we just stop normaly
                $APACHE2CTL stop 2>&1 | grep -v 'not running' >&2 || true
	else
		# if we are here something is broken and we need to try
		# to exit as nice and clean as possible
		PID=$(pidof_apache) || true

		if [ "${PID}" ]; then
			# in this case it is everything nice and dandy and we kill apache2
			echo
			log_warning_msg "The apache2$DIR_SUFFIX configtest failed, so we are trying to kill it manually. This is almost certainly suboptimal, so please make sure your system is working as you'd expect now!"
                        kill $PID
		elif [ "$(pidof apache2)" ]; then
			if [ "$VERBOSE" != no ]; then
                                echo " ... failed!"
			        echo "You may still have some apache2 processes running.  There are"
 			        echo "processes named 'apache2' which do not match your pid file,"
			        echo "and in the name of safety, we've left them alone.  Please review"
			        echo "the situation by hand."
                        fi
                        return 1
		fi
	fi
}

apache_wait_stop() {
	# running ?
	PIDTMP=$(pidof_apache) || true
	if kill -0 "${PIDTMP:-}" 2> /dev/null; then
	    PID=$PIDTMP
	fi

	apache_stop

	# wait until really stopped
	if [ -n "${PID:-}" ]; then
		i=0
		while kill -0 "${PID:-}" 2> /dev/null;  do
        		if [ $i = '60' ]; then
        			break;
        	 	else
        			if [ $i = '0' ]; then
                			echo -n " ... waiting "
        			else
                	      		echo -n "."
        		 	fi
        			i=$(($i+1))
        			sleep 1
        	      fi
		 done
	fi
}

case $1 in
	start)
		log_daemon_msg "Starting web server" "apache2"
		if $APACHE2CTL start; then
			if check_htcacheclean ; then
				log_progress_msg htcacheclean
				start_htcacheclean || log_end_msg 1
			fi
                        log_end_msg 0
                else
                        log_end_msg 1
                fi
	;;
	stop)
		if check_htcacheclean ; then
			log_daemon_msg "Stopping web server" "htcacheclean"
			stop_htcacheclean
			log_progress_msg "apache2"
		else
			log_daemon_msg "Stopping web server" "apache2"
		fi
		if apache_wait_stop; then
                        log_end_msg 0
                else
                        log_end_msg 1
                fi
	;;
	graceful-stop)
		if check_htcacheclean ; then
			log_daemon_msg "Stopping web server" "htcacheclean"
			stop_htcacheclean
			log_progress_msg "apache2"
		else
			log_daemon_msg "Stopping web server" "apache2"
		fi
		if $APACHE2CTL graceful-stop; then
                        log_end_msg 0
                else
                        log_end_msg 1
                fi
	;;
	reload | force-reload | graceful)
		if ! $APACHE2CTL configtest > /dev/null 2>&1; then
                    $APACHE2CTL configtest || true
                    log_end_msg 1
                    exit 1
                fi
                log_daemon_msg "Reloading web server config" "apache2"
		if pidof_apache > /dev/null ; then
                    if $APACHE2CTL graceful $2 ; then
                        log_end_msg 0
                    else
                        log_end_msg 1
                    fi
                fi
	;;
	restart)
		if ! $APACHE2CTL configtest > /dev/null 2>&1; then
		    $APACHE2CTL configtest || true
		    log_end_msg 1
		    exit 1
		fi
		if check_htcacheclean ; then
			log_daemon_msg "Restarting web server" "htcacheclean"
			stop_htcacheclean
			log_progress_msg apache2
		else
			log_daemon_msg "Restarting web server" "apache2"
		fi
		PID=$(pidof_apache) || true
		if ! apache_wait_stop; then
                        log_end_msg 1 || true
                fi
		if $APACHE2CTL start; then
			if check_htcacheclean ; then
				start_htcacheclean || log_end_msg 1
			fi
                        log_end_msg 0
                else
                        log_end_msg 1
                fi
	;;
	start-htcacheclean)
		log_daemon_msg "Starting htcacheclean"
		start_htcacheclean || log_end_msg 1
		log_end_msg 0
	;;
	stop-htcacheclean)
		log_daemon_msg "Stopping htcacheclean"
			stop_htcacheclean
			log_end_msg 0
	;;
	status)
		PID=$(pidof_apache) || true
		if [ -n "$PID" ]; then
			echo "Apache2$DIR_SUFFIX is running (pid $PID)."
			exit 0
		else
			echo "Apache2$DIR_SUFFIX is NOT running."
			if [ -e "$PIDFILE" ]; then
				exit 1
			else
				exit 3
			fi
		fi
	;;
	*)
		log_success_msg "Usage: /etc/init.d/apache2$DIR_SUFFIX {start|stop|graceful-stop|restart|reload|force-reload|start-htcacheclean|stop-htcacheclean|status}"
		exit 1
	;;
esac
```



## SysV Startup Scripts (III)

* Los scripts que se ejecutan en cada nivel se indican en `/etc/rc.d/rc?.d` o `/etc/init.d/rc?.d` o `/etc/rc?.d`
* La `?` indica el nivel

```bash
/etc/rc2.d$ ls
README               S20vboxautostart-service    S50rsync       S98tlp
S16sslh              S20vboxballoonctrl-service  S50saned       S99acpi-support
S20hddtemp           S20vboxdrv                  S70dns-clean   S99grub-common
S20kerneloops        S20vboxweb-service          S70pppd-dns    S99rc.local
S20smartmontools     S20winbind                  S75sudo
S20speech-dispatcher S31atieventsd               S91apache2
S20sysfsutils        S50pulseaudio               S95preload
ijfviana@vagalume:/etc/rc2.d$
```

Note:
The runlevel-specific scripts are stored in /etc/rc.d/rc?.d, /etc/init.d/rc?.d, /etc/rc?.d, or a similar location. (The
precise location varies between distributions.) In all these cases, ? is the runlevel number.



## SysV Startup Scripts (IV)

* El nombre de cada fichero empieza por `S` o `K`
* Un fichero que empieza por `S` indica que el servicio debe arrancar mientras que una `K` indica que debe finalizar

```bash
ijfviana@vagalume:/etc/rc2.d$ ls -l
total 4
-rw-r--r-- 1 root root 677 jul 26  2012 README
lrwxrwxrwx 1 root root  14 mar 19  2013 S16sslh -> ../init.d/sslh
lrwxrwxrwx 1 root root  17 abr 17  2013 S20hddtemp -> ../init.d/hddtemp
lrwxrwxrwx 1 root root  20 mar 14  2013 S20kerneloops -> ../init.d/kerneloops
lrwxrwxrwx 1 root root  23 abr 16  2013 S20smartmontools -> ../init.d/smartmontools
lrwxrwxrwx 1 root root  27 mar 14  2013 S20speech-dispatcher -> ../init.d/speech-dispatcher
lrwxrwxrwx 1 root root  20 jun 19 08:54 S20sysfsutils -> ../init.d/sysfsutils
lrwxrwxrwx 1 root root  31 mar 16  2013 S20vboxautostart-service -> ../init.d/vboxautostart-service
lrwxrwxrwx 1 root root  33 mar 16  2013 S20vboxballoonctrl-service -> ../init.d/vboxballoonctrl-service
lrwxrwxrwx 1 root root  17 mar 16  2013 S20vboxdrv -> ../init.d/vboxdrv
ijfviana@vagalume:/etc/rc2.d$
```

Note:
* When entering a runlevel, rc passes the start parameter to all the scripts with names that begin with a capital S and passes the stop parameter to all the scripts with names that begin with a capital K.
* These SysV startup scripts start or stop services depending on the parameter they’re passed, so the naming of the scripts controls whether they’re started or stopped when a runlevel is entered.
These scripts are also numbered, as in S10network and K35smb; the numbers control the order in which services are started or stopped.
* In reality, the files in the SysV runlevel directories are symbolic links to the main scripts, which are typically
stored in /etc/rc.d, /etc/init.d, or /etc/rc.d/init.d (again, the exact location depends on the distribution).



## SysV Startup Scripts (V)

* El número que sigue a la `S` o `K` indica el orden de ejecución

```bash
/etc/rc2.d$ ls
README                S20vboxautostart-service    S50rsync
S16sslh               S20vboxballoonctrl-service  S50saned
S20hddtemp            S20vboxdrv                  S70dns-clean
S20kerneloops         S20vboxweb-service          S99rc.local
S20smartmontools      S20winbind                  S75sudo
S20speech-dispatcher  S31atieventsd               S91apache2
S20sysfsutils         S50pulseaudio               S95preload
```

Note:
The rc program runs SysV startup scripts in numeric order. This feature enables distribution designers to control
the order in which scripts run by giving them appropriate numbers. This control is important because some
services depend on others.



## Gestionando *scipts* (I)

* Añadir o quitar servicios en un nivel de ejecución basta con borrar o crear los enlaces simbólicos
* El método recomendado es usar las herramientas
 * [chkconfig](http://linux.die.net/man/8/chkconfig)
 * [update-rc.d](http://manpages.ubuntu.com/manpages/hardy/es/man8/update-rc.d.8.html)

Note:
The SysV startup scripts in the runlevel directories are symbolic links back to the original script. This is done so you
don’t need to copy the same script into each runlevel directory. Instead, you can modify the original script without
having to track down its copies in all the SysV runlevel directories. Using a single linked-to file also simplifies
system updates.
You can also modify which programs are active in a runlevel by editing the link filenames. Numerous utility
programs are available to help you manage these links, such as chkconfig, ntsysv, update-rc.d, and rc-update. I
describe chkconfig and update-rc.d tools because they are supported on many distributions. If your distribution
doesn’t support these tools, you should check distribution-centric documentation.



## Gestionando *scipts* (II)
### chkconfig

* Aplicación propia de sistemas RedHat
* Ver los servicios disponible y en que nivel se ejecutan
```bash
$ chkconfig --list
pcmcia
0:off 1:off 2:on 3:on 4:on 5:on 6:off
nfs-common
0:off 1:off 2:off 3:on 4:on 5:on 6:off
xprint
0:off 1:off 2:off 3:on 4:on 5:on 6:off
setserial
0:off 1:off 2:off 3:off 4:off 5:off 6:off
```
* Preguntar por un servicio determinado
```bash
$ chkconfig --list nfs-common
nfs-common
0:off 1:off 2:off 3:on 4:on 5:on 6:off
```

Note:
To list the services and their applicable runlevels with chkconfig, use the --list (or, usually, -l) option. The output
looks something like this but is likely to be much longer:
$ chkconfig --list
pcmcia
0:off 1:off 2:on 3:on 4:on 5:on 6:off
nfs-common
0:off 1:off 2:off 3:on 4:on 5:on 6:off
xprint
0:off 1:off 2:off 3:on 4:on 5:on 6:off
setserial
0:off 1:off 2:off 3:off 4:off 5:off 6:off
This output shows the status of the services in all seven runlevels. For instance, you can see that nfs-common is
inactive in runlevels 0–2, active in runlevels 3–5, and inactive in runlevel 6.
On Red Hat, Fedora, and some other distributions, chkconfig can manage servers that are handled by
scripts. The xinetd-mediated servers appear at the end of the chkconfig listing.
xinetd as well as SysV startup
If you’re interested in a specific service, you can specify its name:
# chkconfig --list nfs-common
nfs-common
0:off 1:off 2:off 3:on 4:on 5:on 6:off



## Gestionando *scipts* (III)
### chkconfig
* Para añadir un *script* a un nivel de arranque
```bash
$ chkconfig --level 23 nfs-common on
```
* Para quitar el script de un nivel de arranque
```bash
$ chkconfig --level 23 nfs-common off
```
* Para que un servicio puede ser gestionado mediante `chkconfig` hay que ejecutar
```bash
$ chkconfig --add nfs-common
```

Note:
To modify the runlevels in which a service runs, use a command like this:
# chkconfig --level 23 nfs-common on
The previous example is for Debian-based systems. On Red Hat and similar systems, you would probably want to target runlevels 3, 4, and
5 with something like --level 345 rather than --level 23.
You can set the script to be on (to activate it), off (to deactivate it), or reset (to set it to its default value).
If you’ve added a startup script to the main SysV startup script directory, you can have chkconfig register it and
add appropriate start and stop links in the runlevel directories. When you do this, chkconfig inspects the script for
special comments to indicate default runlevels. If these comments are in the file and you’re happy with the
suggested levels, you can add it to these runlevels with a command like this:
# chkconfig --add nfs-common
This command adds the nfs-common script to those managed by chkconfig. You would, of course, change nfs-
common to your script’s name. This approach may not work if the script lacks the necessary comment lines with
runlevel sequence numbers for chkconfig’s benefit.



## Gestionando *scipts* (IV)
### update-rc.d (I)

* Se usa en versiones derivadas de Debian
* Para crear los enlaces de arranque en un determinado nivel
```bash
update-rc.d apache2 start 2345
```
* Para crear los enlaces de finalizacion en un determinado nivel
```bash
update-rc.d apache2 stop 016
```
* Para habilitar los enlaces de arranque
```bash
update-rc.d apache2 enable 2345
```



## Gestionando *scipts* (V)
### update-rc.d (II)

* Para deshabilitar los enlaces
```bash
update-rc.d apache2 disable 2345
```
* Borrar los enlaces asociados a un servicio
```
update-rc.d -f apache2 remove
```
* Añadir un servicio en el nivel por defecto (2,3,4,5 - 0,1 6)
```bash
update-rc.d apache2 defaults
```
* Añadir un servicio en el nivel por defecto con una prioridad
```
update-rc.d apache2 defaults 91
```
* Añadir un servicio en el nivel por defecto con una prioridad de arranque y otra de finalización
```
update-rc.d apache2 defaults 20 80
```

Note:
* The update-rc.d program is most common on Debian and some of its derived distributions.
Its basic syntax is: update-rc.d [options] name action
* -n, which causes the program to report what it would do without taking any real action. The name is the name of the service to be modified.
* The action is the name of the action to be performed, along with any action-specific options.
 * remove: Removes links in runlevel-specific directories to the named service. The service’s main script must not exist. This option is
        intended to clean up the SysV startup scripts after a service has been completely removed from the system.
 * defaults: Creates links to start the service in runlevels 2, 3, 4, and 5, and to stop it in runlevels 0, 1, and 6.
 * start NN: Creates a link to start the service in the specified runlevels, using the sequence number NN.
 * stop NN Creates links to stop the service in the specified runlevels, using the sequence number NN.
 * enable [runlevel]: Modifies existing runlevel links to enable the service in the specified runlevel. If no runlevel is specified, runlevels 2, 3, 4, and 5 are modified.
 * disable Modifies existing runlevel links to disable the service in the specified runlevel. If no runlevel is specified, runlevels 2, 3, 4, [runlevel] and 5 are modified.
As an example of rc-update.d in action, consider the following two commands:
# update-rc.d samba defaults
# update-rc.d gdm disable 234
The first of these examples sets the Samba server to run in the default runlevels. The second causes the GNOME
Display Manager (GDM) login server to not run in runlevels 2, 3, and 4
