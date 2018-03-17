# apuntes sesion 2

Veamos qué información obtenemos de la orden lsblk.
Vemos que tenemos 2 discos, sda y sba, cada una con una partición. Dentro tenemos nuestros volúmenes lógicos, tres de ellos cifrados. La semana que viene veremos qué es un volumen lógico cifrado activado.

Queremos nuestro centOS y ubuntu instalado en una red local con el anfritrión.
Con ubuntu vamos a añadir una interfaz de red.

Con crtl+w abrimos el administrador de red anfitrión. En este caso ya teníamos uno creado así que usaremos eso.
Configuración, red, adaptador 2, adaptardor solo anfritrión y seleccionamos la que hemos visto antes.

Probando ifconfig comprobamos que no aparece la interfaz ya que no está configurada.
Arrancamos la máquina y vamos a configurar la interfaz de red que acabamos de insertar.
La configuración se realiza en el archivo /etc/network/interfaces que vamos a editar. Haciendo lspci vemos donde está ubicada. Ahora editamos en el archivo interfaces para que la monte automáticamente.
Primero hacemos una copia del archivo por lo que pudiera pasar. cp -a /etc/network/interfaces /etc/network/interfaces.old
Tenemos que añadir al archivo intefaces

auto enp0s8
iface enp0s8 inet static
  adress 192.168.56.105

y actualizamos con ifup enp0s8 y la interfaz debe estar configurada. Con ifconfig podemos comprobarlo.

Para modificar archivos la mejor opción es Vi. inserción (i), undo (u). Las maneras de salir son :wq (escribe y sale), :q (salir sin escribir), :q(salir sin escribir para salir sin guardar modificaciones, no te da aviso)

Apagamos ubuntu y pasamos a la configuración de centOS.

Con el comando lsblk vamos a ver la instalación por defecto de centOS. Vemos dos particiones sda1 y sda2.
Tenemos una partición con boot de un tamaño de 1GB, claramente sobredimensionada. Tenemos que añadir la interfaz de red editando el archivo de configuración. hacemos cd /etc/sysconfig/network-scripts/. Tenemos un archivo para la interfaz enp0s3. Lo modificamos y cambiamos ONBOOT=no a yes. Lo copiamos en enp0s8.
Modificamos este archivo.

BOOTPROTO=none
NAME=enp0s8
DEVICE=enp0s8
ONBOOT=yes
IPADDR=192.168.56.110
NETMASK=255.255.255.0

Eliminamos el resto de líneas (d en vi).

Tomamos una instantánea en Virtualbox para poder volver a este estado del sistema en cualquier momento.

Es muy probable que /var nos de problemas porque necesite más espacio o prestaciones. Por ello vamos a querer montar /var en otro dispositivo de memoria mejor. Veamos como hacerlo. CentOS utiliza xfs como sistema de archivos.

Utilizamos el comando df -h (comprbar para qué).

lvmdiskscan sirve para ver cómo se están usando los volúmenes lógicos. lvs muestra un resumen de los volumenes lógicos. lvmdisplay (ver qué hace).

Vamos a ñadir el disco y procedemos a la configuración.

Los pasos que vamos a seguir van a ser:
1. Crear el volumen físico (phisical volume)
2. Añadimos el volumen físico al grupo de volumenes (volume group). A esto se le llama extender el grupo de volúmenes.
3. Creamos el volumen lógico (logical volume)
4. Creamos el sistema de archivos
5. Cambiar el punto de montaje

Para crear el phisical volume utilizamos el comando pvcreate pasándole como parámetro /dev/sdb. Para comprobar este proceso usamos pvdisplay que muestra información de los phisical volume del sistema.

Para comprobar información del volume group usamos el comando vgdisplay. Vamos a añadir el phisical volume al volume group, extender el volumegroup. Para ello usamos vgextend cl /dev/sdb (cl es el nombre del grupo de volúmenes

Vamos a crear el volumen lógico con lvcreate -L 4G -n newvar cl, siendo -L el tamaño, -n el nombre y cl el grupo de volúmenes al que pertenece. Con lvdisplay podemos ver la información sobre los volúmenes lógicos del sistema.

Nos queda crear el sistema de archivos. Para ello usamos el comando mkfs -t ext4 /dev/cl/newvar siendo -t el tipo del sistema.
Para acceder al sistema de archivos tenemos que montarlo. Seguiremos los siguientes pasos:

1. Montar el sistema de archivos haciendo un mkdir /media/newvar
2. mount /dev/mapper/cl-newvar /media/newvar
3. (es importante modificar el runlevel para evitar que otros procesos interfieran en este, con systemctl isolate runlevel1.target) systemctl isolate runlevel1.target . También tenemos que tener cuidado con SELinux (security enhanced linux). Para ello hay que añadir a cp la opción --preserve=context . Con cp -a /var/. /media/newvar estamos preservando el contexto y además aplicando la recursividad de -r.


Para comprobar que está montado podemos utilizar mount.

Por último nos queda indicar al sistema operativo, editando el fstab, que /var ahora está en /media/newvar. Tenemos que escribir en el archivo

/dev/mapper/cl-newvar /var  ext4  default 0 0

Previamente habremos creado una copia del archivo por si a caso cpn cp /etc/fstab /etc/fstab.original
Después montamos con mount -a y unmount /media/newvar
Antes de montar, hacemos mv /var /var_old para guardar el antiguo /var por si hubiera errores.

NOTA: HA FALTADO DESMONTAR EL ANTIGUO VAR.
