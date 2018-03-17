# PRÁCTICAS

## Práctica 1

¿Por qué usamos Linux?
- Mayor mercado frente a Windows
- Mayor facilidad para virtualización (menor coste)

¿Por qué Ubuntu y CentOS?
- Ubuntu es la imagen más descargada y usada.
- CentOS basado en Red Hat, gran estabilidad. Una de las imágenes más instaladas. Actualmente CentOS pertenece a Red Hat.

¿Qué es un RAID?
Redundant Array of Independent Discs. Se utilizan para conseguir mayor productividad y mayor fiabilidad.

RAID 0
RAID 1 (Mirroring)

Implementaciones: RAID hardware y RAID software.

- Raid hardware: Independiente de SO. Más eficiente. Mayor coste, dependencia del fabricante. Mayor coste en humanware. Mayor fiabilidad
- Raid software:

¿Cómo tenemos configuradas las máquinas virtuales?
En el Eanfitrión se encuentran dos máquinas virtuales

LVM: logical volume manager. Permite crear abstracciones de volúmenes físicos de memoria y redimensionarlas fácilmente.

Almacenaiento real /discos < Volúmenes físicos < Grupos de volúmnees < Volúmenes lógicos

Se utiliza LVM para poder realizar ajustes de diseño después de haberte creado los volúmenes.

Vamos a crear la máquina virtual de UbuntuServer. Necesitamos un RAID1 por lo que necesitamos otro disco duro virtual para la máquina virtual. Instalamos ubuntu, contraseña PRACTICAS,ISE
No ciframos la carpeta personal. Queremos FDE (full disc encription)

Particionado del disco manual. Tenemos que crear una tabla de particiones para poder continuar. Para crear una tabla de particiones para cada disco nos colocamos sobre el disco, intro y creamos la tabla de particiones.
Lo hacemos para ambos discos. EMpezamos a hacer la configuración de nuestro raid por software. Vamos a crear un dispositivo MD RAID1. Número de dispositivos 2. Número de dispositivos libres 0. Seleccionamos con espacio SDA y SDB y continuamos. Le damos a Sí y terminamos la creación de nuestro RAID1.

Pasamos a configurar LVM. Crear grupo de volúmenes. Le llamamos VG (por ejemplo) y continuamos. Espacio para seleccionar sobre qué dispositivo físico queremos. Trabajamos con el RAID1 creado. TEnemos pues un grupo de volúmenes que está utilizando un volumen físico (RAID1 configurado por 2 volumenes).
Una vez que tenemos un grupo de volúmenes. Vamos a empezar a crear los volúmenes lógicos. Creamos un volumen lógico en nuestro grupo de volúmenes VG. El primero será el arranque. Lo llamamos boot, y 200MB de tamaño. Creamos otro volumen lógico para el usuario, home con 1GB  de tamaño. Creamos otro volumen lógico raiz con el resto de memoria. Creamos otro para swap con 1 o 2 GB. Para swap se suele coger de tamaño el doble de la RAM.
Terminamos y vamos a cifrar los volúmenes. Seleccionamos crear volúmenes cifrados. Con la versión actual de GRUB no podemos tener un FDE por lo que no podemos cifrar boot. El resto sí lo ciframos. Seleccionamos pues todos menos boot. (Si no ciframos swap se producirá un error. Probar). Seleccionamos la información del cifrado por defecto. Seleccionamos finish. La frase de paso será PRACTICAS,ISE. La tenemos que introducir 6 veces, 2 por cada volumen.

Nos ponemos sobre el volumen lógico que queremos utilizar. En este caso boot. Utilizar como. Seleccionamos el sistema de archivos ext2. Punto de montaje /boot. Opciones de montaje por defecto. Buscar qué es bloque reservado. Uso habitual standard. Hemos acabado de definir la partición boot. Hacemos lo mismo con el resto cambiando el punto de montaje. Con home seleccionas /home. Cuidado. No escribir en el cifrado, sino en el sistema de archivos.
Cuando nos pregunten por proxy, como no queremos, lo dejamos en blanco. Las actualizaciones automáticas seleccionamos que no (pensar por qué). Por último, de los servicios que nos vienen por defecto dejaremos solo standard system utilities y continuar. Por último tenemos que instalar el gestor de arranque en disco. Nos preguntará que si en sda o sdb. Seleccionamos sda. POdemos comprobar los bloques con lsblk.
