# 🛡️ Script de Backups Automáticos con Disco Inmutable y Rsync

Este script en bash automatiza el proceso de backup local y remoto, protegiendo los archivos con inmutabilidad (`chattr`) y manteniendo una rotación automática de copias.

## 📦 ¿Qué hace?

- Desactiva temporalmente la protección inmutable del disco
- Comprime los datos de un directorio fuente (`tar`)
- Mueve el backup a una unidad protegida
- Mantiene un máximo de 10 backups (los más antiguos se eliminan)
- Reactiva la protección inmutable
- Borra el backup anterior en un servidor remoto
- Envía los datos actualizados por `rsync`
- Guarda un log completo de todo el proceso

---

## ⚙️ Variables que debes configurar

Edita el script antes de usarlo y cambia estas rutas según tu sistema:

```bash
#!/bin/bash

#Vatiables:
dnas="/ruta_carpeta_nas"
dcopias="/ruta_disco_copias"

#Sacar logs a un mismo archivo
exec > >(tee -a "log.txt") 2>&1

###################################################################

#Copias comprimidas en local

#Quitar Directorio inmutable
chattr -i $dcopias  && echo "Deshabilitado directorio inmutable"

#Mover al directorio del nas
cd $dnas && echo "Accediendo al directorio"

#Comprimir por fecha datos del nas
tar -czvf $(date +%Y-%m-%d).tar.gz $dnas/* 2>&1 && echo "Archivos comprimidos"

#Mover datos comprimidos a unidad de copias local
mv *.tar.gz $dcopias && echo "Datos movidos a la unidad inmutable"

#Borrar archivos mas antiguos con un maximo de 10 Slots
ls -1t $dcopias/*.tar.gz | tail -n +11 | xargs rm --  2>&1 && echo "archivo con antiguedad de mas 10 dias borrado"

#Añadir  Directorio inmutable
chattr +i $dcopias && echo "Habilitado directorio inmutable"

#Mover al /root
cd /root && echo "accediendo a la carpeta root"

####################################################################

#Copias a maquina Rsync#

#Borrado de archivos
ssh user@IP_Servidor "rm -r /backups/* " && echo "Borrado el repositorio remoto"

#Copia a la maquina de rsync
rsync -ravz $dnas/* user@IP_Servidor/ruta && echo "Datos copiados correctamente"
