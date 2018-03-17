# apuntes sesion 3

En esta sesión vamos a crear un raid 1 manualmente en centOS para guardar /var. En la siguiente sesión cifraremos la información de este dispositivo.

Creamos una nueva máquina centOS por defecto y vamos a añadir dos discos duros, ya que queremos un RAID 1 exclusivamente para /var. Comprobamos que están los discos con lslbk.
Para crear el raid 1 usaremos la orden mdadm. No está isntalado por defecto, así que tendríamos que hacer yum install mdadm. Si nos da error es porque no tenemos conexión a internet. Hay que introducir
ifup enp0s3. Podemos hacer un ping a google para ocmprobar si tenemos. Entonces sí podemos hacer yum install mdadm.

mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc Al ejecutar esta línea nos va a decir que no tenemos espacio para almacenar metadatos, ya que no les hemos dado a los discos una tabla de particiones.
Como no nos son necesarios para lo que queremos, podemos continuar. Escribimos yes y tenemos ya creado nuestro raid. Podemos comprbarlo con lsblk.

Creamos el volumen físico con pvcreate. Puesto que queremos que el volumen físico cubra el raid, lo creamos con md0.
pvcreate /dev/md0
Podemos comrpobar que se ha realizado correctamente con pvs o pvdisplay.
A continuación tendremos que crear un grupo de volúmenes con vgcreate, pasándole el nombre del grupo de volúmenes y el volumen físico.
vgcreate pmraid1 /dev/md0
Comprobamos que la creación es correcta con vgs o vgdisplay.
Creamos el volumen lógico con lvcreate -L longitud -n nombre grupo_de_volumenes
lvcreate -L 1G -n newvar pmraid1

Nos ponemos en modo
systemctl isolate runlevel1.target

Para crear el sistema de archivos

mkfs -t xfs /dev/mapper/pmraid1-newvar

Vamos a crear el sitio donde vamos a almacenar temporalmente el contenido del var actual.

mkdir /media/newvar
mount /dev/mapper/pmraid1-newvar /media/newvar

comprobamos que está correcto con mout | grep var
Hacemos cp -a /var/. /media/newvar/ (investigar por qué ponemos . y no * )

Editamos el archivo /etc/fstab añadiendo la linea
/dev/mapper/pmraid1-newvar /var xfs defaults 0 0

Hacemos la copia de seguridad pertinente

mkdir /var_secu
mv /var/. /var_secu/
mv /var /var_old
restorecon /var
mount -a

Comprobamos que el procedimiento es correcto con lsblk.

Si al hacer el mv da un error diciendo que el dispositivo está ocupado. Esto es debido a que algún proceso está accediendo a /var. Por lo tanto habría que terminar el proceso que esté accediendo a /var. Podemos comprobarlo con las órdenes lsof o fuser.

Desmontamos /media/newvar
umount /media/newvar

Y comprobamos que todo está bien con
df -h

Ahora podríamos probar el raid1 quitando uno de lso discos y comprobando que la máquina funciona.

Ahora queremos cifrar /var. Antes crearemos una instantánea de la máquina para recuperarla si surgieran problemas.

Vamos a cifrar con LUKS. Existen dos alternativas, cifrar el dispositivo y usar LVM encima o usar LVM y cifrar los volúmenes lógicos. Nosotros usaremos la segunda opción.
Tenemos que instalar cryptsetup
yum install -y cryptsetup
Recordar que para instalar tenemos que salir del modo de rescate con Crtl+D.

Como la información de /var se va a sobreescribir, tendremos que llevarnos la información a otro sitio. Lo debemos hacer con modo de rescate.
mkdir /varRAID
cp -a /var/. /varRAID/

Ya tenemos la copia. Procedemos al cifrado del volumen lógico.
cryptsetup luksFormat /dev/mapper/pmraid1-newvar
Antes de esto deberíamos hacer un shred a pmraid1 para evitar problemas de seguridad (rellenar de basura).

Le decimos yes y verificamos la contraseña.
Borramos en fstab la línea de /var para evitar el error que nos sale. Etonces /var se borrará en /
mount -a
unalias cp
cp -a /varRAID/. /var

Comprobamos con mount | grep var que sigue /dev/mapper/pmraid1-newvar sigue montado, aunque no debería. Esto no tiene por qué pasar. No debería ser así, ya que hemos intentado desmontarlo.
Utilizaremos

lsof /var

para comprobar qué proceso está usando /dev/mapper/pmraid1-newvar y así poder acabarlo para poder desmontarlo correctamente.
kill -9 (identificador de proceso)

Y ahora sí funcionará correctamente
umount /dev/mapper/pmraid1-newvar

Comprobamos con mount | grep var que ya no está montado.

Copiamos el contenido de /var antiguo al nuevo.

cp -a /varRAID/. /var

Ahora sí ejecutamos
cryptsetup luksFormat /dev/mapper/pmraid1-newvar

Si hacemos mount /dev/mapper/pmraid1-newvar /media/newvar veremos que no sda un error porque está cifrado.

open es el equivalente a mount para sistemas cifrados.
Tenemos nuestro volumen cifrado. Primero tendremos que abrirlo con luksOpen y luego montarlo. Antes de montarlo habrá que crear el makefilesystem porque al cifrarlo hemos perdido el sistema de archivos.

cryptosetup luksOpen /dev/mapper/pmraid1-newvar pmraid1-newvar_crypt

Introducimos la contraseña y con lsblk podemos comprobar que tenemos newvar_crypt. Vamos a intentar montarlo, aunque no nos dejará. Tenemos que hacer el sistema de archivos.

mkfs -t xfs /dev/mapper/pmraid1-newvar_crypt

finalmente, ya podemos hacer el mount

mount /dev/mapeer/pmraid-newvar_crypt /media/newvar

Vamos a comprobar que está montado con

mount | grep var y lsblk

A continuación copiamos nuestro var al nuevo var

cp -a /var/. /media/newvar

Y vamos a deditar el fstab con la línea

/dev/mapper/pmraid1-newvar_crypt  /var  xfs defaults  0 0

Y hacemos un

mv /var /var_borrame
mkdir /var
restorecon /var

Si hacemos mount -a funciona correctamente, pero si reiniciamos, perderemos el cryptsetup luksOpen. Si queremos que el sistema lo haga cada vez que se inicie hay que editar el archivo
/etc/crypttab, en el que tendremos que hacer referencia al UUID del dispositivo. Para comprobarlo lo haremos con blkid | grep crypto. Veremos que hay 2 UUID, uno para el activado y otro del sin activar.
Elegiremos el sin activar. La línea que hay que agregar al crypttab es, por tanto

NombreDestinoUnaVezAbierto UUID none

en este caso

pmraid1-newvar_crypt UUID=... none

Podemos hacerlo como

blkid | grep crypto >> etc/crypttab

y después editar el archivo para que quede tal y como queremos. De esta manera nos ahorramos copiar el UUID a mano. Así tenemos el crypttab listo. Hecho esto hemos terminado el proceso. Salimos del modo de rescate con systemctl isolate default.target. Reiniciamos y debe funcionar.
Nos pedirá la contraseña para el encriptado. Tras introducirla debe funcionar el sistema operativo.
