## ¿Qué son?

<a class="fancybox" href="img/nivel_de_arranque.png" data-fancybox-group="gallery" title="Nivel de arranque">
<img height="550px" src="img/nivel_de_arranque.png" alt="Niveles de arranque">
</a>

Note:
El término runlevel o nivel de ejecución se refiere al modo de operación en los sistemas operativos que implementan el estilo de sistema de arranque de iniciación tipo UNIX System V.



## ¿Cuántos niveles hay? (I)

Los 7 niveles de ejecución (runlevels) estándars

<div class="table-responsive">
  <table class="table table-hover table-condensed table-bordered">
    <thead>
    <tr>
    <th>Nivel</th>
    <th>Nombre</th>
    <th>Descripción</th>
    </tr>
    </thead>
    <tbody>
    <tr>
    <td>0</td>
    <td>Alto</td>
    <td>Alto o cierre del sistema (Apagado).</td>
    </tr>
    <tr>
    <td>1,s,S</td>
    <td>Monousuario</td>
    <td>No configura la interfaz de red ni los demonios de inicio, sólo permite el acceso al root. Permite reparar problemas, o hacer pruebas en el sistema.</td>
    </tr>
    <tr>
    <td>2</td>
    <td>Multiusuario</td>
    <td>Multiusuario sin soporte de red.</td>
    </tr>
    <tr>
    <td>3</td>
    <td>Multiusuario</td>
    <td>Multiusuario con soporte de red.</td>
    </tr>
    </tbody>
  </table>
</div>

Note:
* 0 : A transitional runlevel, meaning that it’s used to shift the system from one state to another. Specifically, it shuts down the system. On modern hardware, the system should completely power down. If not, you’re expected to either reboot the computer manually or power it off.
* 1, s, or S:  Single-user mode. What services, if any, are started at this runlevel varies by distribution. It’s typically used for low-level system maintenance that may be impaired by normal system operation, such as resizing partitions. Typically, s or S produces a root shell without mounting any filesystems, whereas 1 does attempt to mount filesystems and launches a few system programs.
* 2 On Debian and its derivatives (including Ubuntu), a full multi-user mode with X running and a graphical login. Most other distributions leave this runlevel undefined.
* 3 On Fedora, Mandriva, Red Hat, and most other distributions, a full multi-user mode with a console (non-graphical) login screen.



## ¿Cuántos niveles hay? (II)

Los 7 niveles de ejecución (runlevels) estándars

<div class="table-responsive">
  <table class="table table-hover table-condensed table-bordered">
<thead>
<tr>
<th>Nivel</th>
<th>Nombre</th>
<th>Descripción</th>
</tr>
</thead>
<tbody>
<tr>
<td>4</td>
<td>Multiusuario con soporte de red</td>
<td>Igual que el 3. (Configurable)</td>
</tr>
<tr>
<td>5</td>
<td>Multiusuario gráfico (X11)</td>
<td>Similar al nivel de ejecución 3 + display manager.</td>
</tr>
<tr>
<td>6</td>
<td>Reinicio</td>
<td>Se reinicia el sistema.</td>
</tr>
</tbody>
</table>
</div>

Note:

* 4 Usually undefined by default and therefore available for customization.
* 5 On Fedora, Mandriva, Red Hat, and most other distributions, the same behavior as runlevel 3 with the addition of having X run with an XDM (graphical) login.
* 6 Used to reboot the system. This runlevel is also a transitional runlevel. Your system is completely shut down, and then the computer reboots automatically.
* Don’t configure your default runlevel to 0 or 6. If you do, your system will immediately shut down or reboot once it finishes powering up.
* Runlevel 1 could conceivably be used as a default, but chances are you’ll want to use 2, 3, or 5 as your default runlevel, depending on your distribution and use for the system.
* Many of the files and file locations described in this chapter are based on the Linux Standards Base (LSB), which is a specification of various standards for the locations of files, the existence of particular libraries, and so on. The LSB is designed to ensure a minimal level of compatibility across common Linux distributions.
