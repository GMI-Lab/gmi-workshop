# Análisis de datos genómicos obtenidos a partir de tecnologías de secuenciación masiva: ensamblaje y análisis de un genoma bacteriano.

## 1.- Introducción
Bienvenidas y Bienvenidos al repositorio de nuestro primer taller del curso, a continuación recibirán una serie de instrucciones para la ejecución de ciertas operaciones en base a líneas de comandos y las instrucciones respectivas.

De aquí en adelante se establecerá el siguiente formato:

[Formato 1]
```
En este tipo de cuadros encontrará comandos, los que podrían tener que ser modificados según los datos de los distintos grupos.
```
[Formato 2]

***En este tipo de sección encontrará preguntas para guiar el análisis de los resultados**

## 2.- Configuraciones iniciales
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
## 3.- Análisis de calidad y filtrado de lecturas
Utilizaremos como ejemplo el resultado de secuenciación de la cepa de *Escherichia coli* C41 previamente publicadas (https://zenodo.org/record/940733#.Ybkgqr1ByCg). El set consiste enlecturas provenientes de secuenciación Illumina apareadas y otro proveniente de secuenciación con tecnología Nanopore. 

En el caso de las lecturas Illumina, utilizaremos FastQC para evaluar las lecturas e identificar problemas comunes asociados a este tipo de tecnología. Para ejecutar el programa utilice el siguiente comando (Notar que el “&” al final del comando permite que la terminal quede disponible para nuevos comandos.):
```
fastqc &
```
Se desplegará una ventana. Hacer clic en “File” y luego navegar hasta encontrar las lecturas Illumina. Puede cargar ambos archivos en la misma instrucción. Luego, el programa hará los cálculos y análisis para cada uno de los archivos.

***Cómo describiría el conjunto de lecturas en cuanto al número y largo de las lecturas. Revise también la calidad de las lecturas y algunos de los indicadores de calidad presentados en el programa. ¿Qué podría hacer para mejorar la calidad de la población de lecturas?***

Basado en lo observado sobre la calidad de las lecturas, utilice Trimmomatic para limpiar las secuencias de las regiones con peores calidades. Para ello el comando tiene la siguiente estructura. Notar que en este comando, y los que siguen en esta guía, los nombres de archivos a reemplazar por los propios están indicados entre llaves ( **{}** ), no debe dejar estos caracteres al hacer el reemplazo.
```
trimmomatic PE {input_FW} {imput_RV} \
{output_FW_pareadas} {output_RV_pareadas} \
{output_FW_NoPareadas} {output_RV_ NoPareadas } \
SLIDINGWINDOW:XX:YY LEADING:ZZ TRAILING:GG MINLEN:BB
```
El comando indica primero que las lecturas son pareadas (*PE*) y luego se indican ambos archivos con las lecturas (*input_FW* e *input_RV*). Los siguientes cuatro parámetros son los archivos en los que las lecturas que son procesadas y que consiguen superar el umbral de tamaño mínimo definido son organizadas. En dos archivos son organizadas las lecturas en los que superan los filtros ambas lecturas del par (*output_FW_paired* y *output_RV_paired*) y los siguientes archivos para las lecturas que pierden su contraparte del par luego del procesamiento (*output_FW_unpaired* y *output_RV_unpaired*). Finalmente, luego de cada filtro se indican los valores a considerar:

- **SLIDINGWINDOW:XX:YY** --> Se configura una ventana que recorre la lectura, eliminando una región que esté por debajo de un umbral de calidad. XX representa el largo de bases de la ventana (generalmente 4) e YY representa la calidad mínima para que una ventana no sea eliminada (generalmente cercana a 23, depende de la calidad de las lecturas.

- **LEADING:ZZ** --> Remueve las primeras bases si tienen una calidad bajo ZZ (generalmente 3).

- **TRAILING:GG** --> Remueve las últimas bases si tienen una calidad bajo GG (generalmente 3).

- **MINLEN:BB** --> Elimina las lecturas que, luego del procesamiento, tengan un largo menor a BB (generalmente 35 pb).

***Construya un comando para Trimmomatic y justifique sus decisiones basado en la calidad de las lecturas. Utilizando FastQC, compare las estadísticas de sus lecturas antes y después del filtro de calidad.***

Finalmente, para el ensamblaje del genoma bacteriano se pueden utilizar las lecturas desapareadas, para ello podemos reunirlas en un solo archivo con el siguiente comando:
```
cat {output_FW_NoPareadas} {output_RV_NoPareadas} > {lecturas_NoPareadas}
```
Ahora, realizaremos el procedimiento análogo para las lecturas Nanopore. Primero, debemos quitar los adaptadores que se unieron a las secuencias de DNA cuando se construye la biblioteca que se secuenció. Para ello utilizamos Porechop con el comando siguiente:
```
porechop -i {lecturas_Nanopore_originales} \
-o {lecturas_Nanopore_SinAdaptadores} \
--threads 8
```
Una vez removidos los adaptadores, utilizaremos NanoPlot para hacer la evaluación de la calidad de las lecturas. El comando se indica a continuación, en el se indica una carpeta donde se escriben los archivos de salida. La opción **--plots dot** indica que los gráficos que se construyen muestran cada lectura como un punto y no hace una aproximación en forma de hexágonos, la que es la opción por defecto.
```
NanoPlot --fastq {lecturas_Nanopore_SinAdaptadores} \
-t 8 --plots dot -o {salida_carpeta}
```
***Analice la calidad de las lecturas de secuenciación Nanopore. Considere la estadística calculada por NanoPlot y algunos de sus gráficos. ¿Aplicaría algún filtro para mejorar la estadística de la población de lecturas?***

Ahora, utilizaremos NanoFilt para hacer filtrado de las lecturas, el comando se indica a continuación. Notar que se hace un pipe entre **cat** y **NanoFilt** para procesar las lecturas. En el comando se indica la calidad promedio de las lecturas mínimas para el filtro (**-q MM**) y el largo mínimo de las lecturas (**-l NN**), los parámetros dependen de la calidad de las lecturas obtenidas.
```
cat EcoliC1_nanopore_woadap.fastq | NanoFilt -q MM -l NN > EcoliC1_nanopore_woadap_filt.fastq
```
***Construya el comando para NanoFilt y justifique sus decisiones basado en la calidad de sus lecturas. Utilizando NanoPlot, compare las estadísticas de sus lecturas antes y después del filtro de calidad.***

## 4.- Ensamblaje
Para el ensamblaje utilizaremos Unicycler. En primer lugar, realizaremos un ensamblaje considerando solo las lecturas Illumina luego del filtrado con el siguiente comando. Los resultados de este ensamblaje se encontrarán finalmente en la carpeta que se indica con la opción **-o**.
```
unicycler -1 {lecturas_FW_pareadas} \
-2 {lecturas_RV_pareadas}           \
-s {lecturas_NoApareadas}           \
-o {carpeta_salida}
```
Repetiremos el ensamblaje, pero ahora incluiremos las lecturas Nanopore con la opción **-l**. Considerar una carpeta para el ensamblaje de lecturas cortas y otra para los resultados del ensamblaje híbrido.
```
unicycler -1 {lecturas_FW_pareadas}
-2 {lecturas_RV_pareadas}          \
-s {lecturas_NoApareadas}          \
-l {lecturas_Nanopore}             \
-o {carpeta_salida}
```
Los contigs que se generen del ensamblaje se encuentran en formato de secuencia de nucleótidos en un archivo FastA, pero también se puede encontrar entre los resultados el mapa que conecta a los contigs usando la información que se incluye al considerar lecturas pareadas. Para visualizar los mapas utilizamos el programa Bandage, el que se llama con el siguiente comando. En la pestaña File buscaremos la opción Load graph y navegaremos hasta encontrar el mapa del ensamblaje. Una vez cargado, hacer clic en la opción Draw graph. Entre la visualización y la estadística que se presenta, se puede tener una idea de que tan completo está el ensamblaje, considerando que la situación ideal es obtener un gran contig circular que represente el cromosoma bacteriano y los plásmidos que le acompañan, eventualmente.
```
bandage &
```
***Compare los mapas obtenidos por ambas estrategias de ensamblaje. ¿Qué impacto tiene la inclusión de las lecturas Nanopore en el resultado del ensamblaje en el caso de su grupo?***

## 5.- Evaluación del ensamblaje
Para evaluar el grado de fragmentación del ensamblaje y la cobertura promedio, utilizaremos la herramienta Quast. A este programa debemos entregarle el resultado del ensamblaje en formato FastA y las lecturas que dieron origen el ensamblaje. Los comandos se presentan a continuación. El informe que reporta Quast es generado en distintos formatos, los que se encontrarán en la carpeta que se le indique con la opción **-o**.
```
quast.py {resultado_ensamblaje}   \
-1 {lecturas_FW_pareadas}         \
-2 {lecturas_RV_pareadas}         \
--single {lecturas_NoPareadas} -o {carpeta_resultados} -t
8
```
Ahora, el comando para analizar el resultado del ensamblaje considerando una estrategia híbrida:
```
quast.py {resultado_ensamblaje}   \
-1 {lecturas_FW_pareadas}         \
-2 {lecturas_RV_pareadas}         \
--single {lecturas_NoPareadas}    \
--nanopore {lecturas_Nanopore} -o {carpeta_resultados} -t
8
```
***Reporte la calidad de sus ensamblajes basadas en las estadísticas calculadas por Quast. Basado en estos resultados, ¿Qué impacto sobre la estadística tuvo la incorporación de las lecturas largas de tecnología Nanopore sobre los resultados del ensamblaje?***

## 6.- Anotación del genoma bacteriano
Utilizaremos la plataforma PATRIC para hacer la anotación general del mejor ensamblaje obtenido. Para ellos cree una cuenta en PATRIC (https://patricbrc.org/). Primero debemos asignar la taxonomía del aislado problema. Para ello, entre los servicios identifique la herramienta Similar Genome Finder, que le permitirá buscar aislados en la base de datos similares al suyo. Cargue su ensamblaje en formato FastA y ejecute la herramienta.

![Readme1](https://user-images.githubusercontent.com/70078596/145698207-0ce8ecaa-4f8d-434c-b6e2-030a3983248b.png)

Con la aproximación de la taxonomía del aislado problema, podemos ejecutar el Comprehensive Genome Analysis. Busque la herramienta entre los servicios y en la
plataforma seleccione que el análisis comience con un genoma ensamblado, carque el archivo FastA con su ensamblaje, busque la taxonomía aproximada de su aislado entre las opciones y seleccione o cree una carpeta para el output.

![Readme2](https://user-images.githubusercontent.com/70078596/145698208-911f8cbc-4376-449f-a7cf-7f07ccc4c6d6.png)

Finalmente, entre los resultados encontrará un resumen en formato HTML. Entre estos datos puede encontrar la estadística de la anotación del genoma, esquemas de los contigs y resultados de la anotación funcional de los genes codificados.

***Analice las estadísticas de la anotación del ensamblaje y analice algunos resultados en cuanto a genes de resistencia y de virulencia reportados.***
