# Taller de Bioinformática 2021: Análisis de datos genómicos obtenidos a partir de tecnologías de secuenciación masiva: ensamblaje y análisis de un genoma bacteriano. Facultad de Ciencias, Universidad de Chile.

## Introducción
Bienvenidos al repositorio de nuestro taller de Bioinformatica, a continuación ustedes van a recibir una seguidilla de protocolos para la ejecución de ciertas funciones y las instrucciones respectivas.

Les adelantamos que de aquí en adelante se establecerá el siguiente formato:

[Formato 1]

```
En este tipo de cuadros encontrará comandos, los que podrían tener que ser modificados según los datos de los distintos grupos.
```
[Formato 2]

***ACTIVIDAD X: En este tipo de cuadro encontrará el detalle de lo que debe reportar del taller. No es necesario incluir una portada en el reporte, en la primera página indique la versión del curso (Bioinformática 2021), el número de grupo y sus integrantes. Para la confección del reporte considere hojas de tamaño carta con fuente Arial o Times New Roman en tamaño 11 puntos e interlineado sencillo. Cada actividad tiene indicada su extensión máxima, información que supere esa extensión no será revisada.***

## Configuraciones iniciales
Utilizaremos en el taller alguna shell de UNIX que se pueda acceder mediante una terminal. Usuarios de Ubuntu o Mac OSX tienen directamente disponible la terminal. En caso de requerirlo, estudiantes pueden acceder de forma remota al servidor ADA del Grupo de Microbiología Integrativa (GMI). Para establecer la conexión vía SSH se utilizarán el usuario y contraseña que se indicarán en el taller. Para la conexión remota, es necesario tener instalada la herramienta de VPN de CISCO (disponible en https://soporte.uchile.cl/articulo/descargar-anyconnect/) y MobaXterm (disponible en https://soporte.uchile.cl/articulo/descargar-anyconnect/).

Si trabaja el taller desde su maquina con UNIX, puede instalar los programas necesarios en ambientes, donde sus dependencias son instaladas separadas del resto del sistema. Para ello, instale miniconda y luego configure bioconda, las instrucciones las encuentra aquí https://bioconda.github.io/user/install.html#install-conda. El siguiente comando creará un ambiente llamado *bactgen* que tendrá instalado los programas a utilizar durante el taller:

```
conda create -n bactgen trimmomatic porechop nanoplot \
fastqc unicycler quast bandage
```
Luego, es necesario activar el ambiente para tener disponibles los programas, para ello ejecute
el siguiente comando:
```
conda activate bactgen
```
Con esto, en el prompt de la terminal debería indicarse que el ambiente está activo de la
siguiente forma:
```
(bactgen)usuario@maquina:~$
```
## Análisis de calidad y filtrado de lecturas
Cada grupo contará con dos sets de lecturas: un set de lecturas provenientes de secuenciación Illumina apareadas (detalles) y otro proveniente de secuenciación con tecnología Illumina. En el caso de las lecturas Illumina, utilizaremos FastQC para evaluar las lecturas e identificar problemas comunes asociados a este tipo de tecnología.

Para ejecutar el programa utilice el siguiente comando (Notar que el “&” al final del comando permite que la terminal quede disponible para nuevos comandos.):
```
fastqc &
```
Se desplegará una ventana. Hacer clic en “File” y luego navegar hasta encontrar las lecturas Illumina de su grupo. Puede cargar ambos archivos en la misma instrucción. Luego, el programa hará los cálculos y análisis para cada uno de los archivos.

***ACTIVIDAD 1: Describa las lecturas entregadas para su grupo en cuanto al número y largo de las lecturas. Describa también la calidad de las lecturas y algunos de los indicadores de calidad presentados en el programa. ¿Qué podría hacer para mejorar la calidad de la población de lecturas? Incluya los gráficos que justifiquen sus observaciones (máximo dos planas).***

Basado en lo observado sobre la calidad de las lecturas, utilice Trimmomatic para limpiar las secuencias de las regiones con peores calidades. Para ello el comando tiene la siguiente estructura. Notar que en este comando y los que vienen en la guía, los nombres de archivos a reemplazar por los propios están indicados entre llaves ( **{}** ), no debe dejar estos caracteres al hacer el reemplazo.
```
trimmomatic PE {input_FW} {imput_RV} \
{output_FW_pareadas} {output_RV_pareadas} \
{output_FW_NoPareadas} {output_RV_ NoPareadas } \
SLIDINGWINDOW:XX:YY LEADING:ZZ TRAILING:GG MINLEN:BB
```
El comando indica primero que las lecturas son pareadas (PE) y luego se indican ambos archivos con las lecturas (input_FW e input_RV). Los siguientes cuatro parámetros son los archivos en los que las lecturas que son procesadas y que consiguen superar el umbral de tamaño mínimo definido son organizadas. En dos archivos son organizadas las lecturas en los que superan los filtros ambas lecturas del par (output_FW_paired y output_RV_paired) y los siguientes archivos para las lecturas que pierden su contraparte del par luego del procesamiento (output_FW_unpaired y output_RV_unpaired). Finalmente, luego de cada filtro se indican los valores a considerar:

- **SLIDINGWINDOW:XX:YY** --> Se configura una ventana que recorre la lectura, eliminando una región que esté por debajo de un umbral de calidad. XX representa el largo de bases de la ventana (generalmente 4) e YY representa la calidad mínima para que una ventana no sea eliminada (generalmente cercana a 23, depende de la calidad de las lecturas.

- **LEADING:ZZ** --> Remueve las primeras bases si tienen una calidad bajo ZZ (generalmente 3).

- **TRAILING:GG** --> Remueve las últimas bases si tienen una calidad bajo GG (generalmente 3).

- **MINLEN:BB** --> Elimina las lecturas que, luego del procesamiento, tengan un largo menor a BB (generalmente 35 pb).

***ACTIVIDAD 2: Indique el comando para Trimmomatic utilizado por su grupo y justifique sus decisiones basado en la calidad de sus lecturas. Utilizando FastQC, compare las estadísticas de sus lecturas antes y después del filtro de calidad. (máximo una plana).***

Finalmente, para el ensamblaje del genoma bacteriano se pueden utilizar las lecturas desapareadas, para ello podemos reunirlas en un solo archivo con el siguiente comando:
```
cat {output_FW_NoPareadas} {output_RV_NoPareadas} \
> {lecturas_NoPareadas}
```
Ahora, realizaremos el procedimiento análogo para las lecturas Nanopore. Primero, debemos quitar los adaptadores unidos a las secuencias de DNA cuando se construye la biblioteca que se secuenció. Para ello utilizamos Porechop con el comando siguiente:
```
porechop -i {lecturas_Nanopore_originales} \
-o {lecturas_Nanopore_SinAdaptadores} \
--threads 8
```
Una vez removidos los adaptadores, utilizaremos NanoPlot para hacer la evaluación de la calidad de las lecturas. El comando se indica a continuación, en el se indica una carpeta donde se escriben los archivos de salida. La opción **--plots dot** indica que los gráficos que se construyen muestran cada lectura como un punto y no hace una aproximación en forma de hexágonos, que es la opción por defecto.
```
NanoPlot --fastq {lecturas_Nanopore_SinAdaptadores} \
-t 8 --plots dot -o {salida_carpeta}
```
***ACTIVIDAD 3: Describa la calidad de las lecturas de secuenciación Nanopore de su grupo. Considere la estadística calculada por NanoPlot y algunos de sus gráficos. ¿Aplicaría algún filtro para mejorar la estadística de la población de lecturas? (máximo una plana).***

Ahora, utilizaremos NanoFilt para hacer filtrado de las lecturas, el comando se indica a continuación. Notar que se hace un pipe entre **cat** y **NanoFilt** para procesar las lecturas. En el comando se indica la calidad promedio de las lecturas mínimas para el filtro (**-q MM**) y el largo mínimo de las lecturas (**-l NN**), los parámetros dependen de la calidad de las lecturas obtenidas.
```
cat EcoliC1_nanopore_woadap.fastq | NanoFilt \
-q MM -l NN > EcoliC1_nanopore_woadap_filt.fastq
```
***ACTIVIDAD 4: Indique el comando para NanoFilt utilizado por su grupo y justifique sus decisiones basado en la calidad de sus lecturas. Utilizando NanoPlot, compare las estadísticas de sus lecturas antes y después del filtro de calidad. (máximo una plana).***

## Ensamblaje
Para el ensamblaje utilizaremos Unicycler. En primer lugar, realizaremos un ensamblaje considerando solo las lecturas Illumina luego del filtrado con el siguiente comando. Los resultados de este ensamblaje se encontrarán finalmente en la carpeta que se indica con la opción **-o**.
```
unicycler -1 {lecturas_FW_pareadas} \
-2 {lecturas_RV_pareadas}           \
-s {lecturas_NoApareadas}           \
-o {carpeta_salida}
```
