## Comprobando el nivel (I)
Podemos comprobar dos cosas:

* [Nivel por defecto al arrancar](#/6/2)
* [Nivel actual](#/6/3)

Note:
Sometimes it’s necessary to check your current runlevel. Typically, you’ll do this prior to changing the runlevel or
to check the status if something isn’t working correctly. Two different runlevel checks are possible: checking your
default runlevel and checking your current runlevel.



## Comprobando el nivel  (II)
### Comprobando el nivel por defecto

* En SysV, el nivel por defecto lo determina la directiva `initdefault` del fichero  `/etc/inittab`:
```bash
$ grep :initdefault: /etc/inittab
id:5:initdefault:
```
* En sistemas Upstart tenemos que editar el fichero `/etc/init/rc-sysinit.conf` y añadir la siguiente linea:
```bash
$ sudo /etc/init/rc-sysinit.conf
env DEFAULT_RUNLEVEL=2
```

Note:
* On a SysV-based system, you can determine your default runlevel by inspecting the /etc/inittab file.
* You may notice that this line does not define a process to run. In the case of the initdefault action, the process field is ignored.
* If you want to change the default runlevel for the next time you boot your system, edit the initdefault line in /etc/inittab and change the runlevel field to the value that you want.
* On a Upstart-based system, You should edit /etc/init/rc-sysinit.conf instead and change the following line: env DEFAULT_RUNLEVEL=2



## Comprobando el nivel (III)
### Comprobando el nivel actual

* La orden [`runlevel`](http://unixhelp.ed.ac.uk/CGI/man-cgi?runlevel+8) sirve para determinar el nivel de arranque actual
```bash
$ runlevel
N2
```
* El número de la izquierda indica el nivel, la letra `N` de la derecha indica que no se ha cambiado de nivel desde el arranque

Note:
If your system is up and running, you can determine your runlevel information with the runlevel command:
# runlevel
N2
The first character is the previous runlevel. When the character is N, this means the system hasn’t switched
runlevels since booting. It’s possible to switch to different runlevels on a running system with the init and telinit
programs, as described next. The second character in the runlevel output is your current runlevel.



## Cambiando el nivel (I)

* [`init` o `telinit`](#/6/5)
* [`shutdown`](#/6/6)
* [`halt`, `reboot` o `poweroff`](#/6/8)

Note:
Sometimes you may want to change runlevels on a running system. You might do this to get more services, such as
going from a console to a graphical login runlevel, or to shut down or reboot your computer. You can accomplish
this with the init (or telinit), shutdown, halt, reboot, and poweroff commands.



## Cambiando el nivel (II)
### init o telinit

* El comando [`init`](http://linux.die.net/man/8/init) se puede usar para cambiar el nivel de arranque actual
```bash
ijfviana@valagume:~$ init 1
N2
ijfviana@valagume:~$ runlevel
S21
ijfviana@valagume:~$ init 6
```
* El comando [`telinit`](http://linux.die.net/man/8/telinit) es similar a [`init`](http://linux.die.net/man/8/init). Su opción `-Q` o `-q` provoca que se relean el fichero `/etc/inittab`

Note:
The init process is the first process run by the Linux kernel, but you can also use it to have the system reread the
/etc/inittab file and implement changes it finds there or to change to a new runlevel. The simplest case is to have it
change to the runlevel you specify. For instance, to change to runlevel 1 (the runlevel reserved for single-user or
maintenance mode), you would type this command:
# init 1
To reboot the system, you can use init to change to runlevel 6 (the runlevel reserved for reboots):
# init 6
A variant of init is telinit. This program can take a runlevel number just like init to change to that runlevel, but it
can also take the Q or q option to have the tool reread /etc/inittab and implement any changes it finds there. Thus,
if you’ve made a change to the runlevel in /etc/inittab, you can immediately implement that change by typing
telinit q.



## Cambiando el nivel (III)
### shutdown

* Permite cambiar de nivel ya que  programa la parada, apagado o reinicio del sistema.

```bash
$ shutdown -r now
```
```bash
$ # shutdown -H +15 "El sistema se parará en 15 minutos"
```
```bash
$ # shutdown -c "Cambio de opinión no se reinicia"
```
```bash
$ # shutdown -P 13:30  "El sistema se apagará a las 13:30"
```

Note:
* It’s better to use the shutdown command in a multi-user environment when you want to reboot, shut down, or switch to single-user mode.
* The shutdown program sends a message to all users who are logged into your system and prevents other users from logging in during the process of changing runlevels.
* The shutdown command also lets you specify when to effect the runlevel change so that users have time to exit editors and safely stop other processes they may have running.
* When the time to change runlevels is reached, shutdown signals the init process for you.



## Cambiando el nivel (IV)
### shutdown

<div class="table-responsive">
  <table class="table">
    <thead>
      <tr>
        <th></th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td></td>
      </tr>
    </tbody>
  </table>
</div>
| Opción | Descripción
|--------|------------------------------------------------------
| `-r`     | Reinicia el sistema
| `-H`     | Para el sistema
| `-P`     | Apaga el sistema
| `-h`     | Apaga o para el sistema (depende de la configuración)
| `-c`     | Cancela la parada planificada



## Cambiando el nivel  (V)

* Otros comando que tienen un funcionamiento similar a  [`shutdown`](http://linux.die.net/man/8/shutdown) son:
 * [`halt`](http://linux.die.net/man/8/halt): para el sistema
 * [`reboot`](http://linux.die.net/man/8/reboot): reinicia el sistema
 * [`poweroff`](http://linux.die.net/man/8/poweroff): apaga el sistema
* Los dos primeros suelen ser enlaces simbólicos al primero

Note:
Three additional shortcut commands are halt, reboot, and poweroff. As you might expect, these commands halt the
system (shut it down without powering it off), reboot it, or shut it down and (on hardware that supports this
feature) turn off the power, respectively. Typically, two of these commands are symbolic links to a third; the single
binary responds differently depending on the name used to call it.
