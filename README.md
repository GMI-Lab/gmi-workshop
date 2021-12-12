# Taller de Bioinformática 2021: Análisis de datos genómicos obtenidos a partir de tecnologías de secuenciación masiva: ensamblaje y análisis de un genoma bacteriano. Facultad de Ciencias, Universidad de Chile.

## Introducción
Bienvenidos al repositorio de nuestro taller de Bioinformatica, a continuación ustedes van a recibir una seguidilla de protocolos para la ejecución de ciertas funciones y las instrucciones respectivas.

Les adelantamos que de aqui en adelante se establecerá el siguiente formato:

[Formato 1]

```
En este tipo de cuadros encontrará comandos, los que podrían tener que ser modificados según los datos de los distintos grupos.
```
[Formato 2]

**Instrucciones:**

**En este tipo de cuadro encontrará el detalle de lo que debe reportar del taller. No es necesario incluir una portada en el reporte, en la primera página indique la versión del curso (Bioinformática 2021), el número de grupo y sus integrantes. Para la confección del reporte considere hojas de tamaño carta con fuente Arial o Times New Roman en tamaño 11 puntos e interlineado sencillo. Cada actividad tiene indicada su extensión máxima, información que supere esa extensión no será revisada.**

## Configuraciones iniciales
Utilizaremos en el taller alguna shell de UNIX que se pueda acceder mediante una terminal. Usuarios de Ubuntu o Mac OSX tienen directamente disponible la terminal. En caso de requerirlo, estudiantes pueden acceder de forma remota al servidor ADA del Grupo de Microbiología Integrativa (GMI). Para establecer la conexión vía SSH se utilizarán el usuario y contraseña que se indicarán en el taller. Para la conexión remota, es necesario tener instalada la herramienta de VPN de CISCO (disponible en https://soporte.uchile.cl/articulo/descargar-anyconnect/) y MobaXterm (disponible en https://soporte.uchile.cl/articulo/descargar-anyconnect/).

Si trabaja el taller desde su maquina con UNIX, puede instalar los programas necesarios en ambientes, donde sus dependencias son instaladas separadas del resto del sistema. Para ello, instale miniconda y luego configure bioconda, las instrucciones las encuentra aquí https://bioconda.github.io/user/install.html#install-conda. El siguiente comando creará un ambiente llamado *bactgen* que tendrá instalado los programas a utilizar durante el taller.
