---

​---
# Materia: Introducción a Unix y R en bioinformática.

## Proyecto final - Reporte de comandos

### Objetivo del proyecto:

Evaluar la aplicación de comandos en Unix y R para la exploración y el análisis de datos bioinformáticos con la finalidad de responder una pregunta de investigación concreta.

### Datos utilizados:

Datos descargados de la colección de Sitios de Unión a Factores de Transcripción del catálogo de experimentos depositados en RegulonDB, basados en la secuenciación acoplada a precipitación de cromatina (ChIP-seq). Específicamente, datos obtenidos en la fase de "peak calling" del algoritmo tradicional de ChIP-seq correspondiente a los sitios de unión al factor de transcripción "Lrp" de _E. coli_.

---

## Descripción General del proyecto

#### Contexto:

La Proteína Reguladora de la respuesta a Leucina (Lrp) es un factor de transcripción procariota, ampliamente estudiado en _E. coli_, relacionado con la regulación del metabolismo de aminoácidos, el transporte de nutrientes, el crecimiento y la dinámica topológica del DNA; todo esto en respuesta a concentraciones cambiantes de leucina (http://regulondb.ccg.unam.mx/regulon?term=ECK120011383&organism=ECK12&type=regulon).

#### Pregunta de investigación:

*¿Existen cambios en los sitios de unión a Lrp en el genoma de _E. coli_ en respuesta a modificaciones en las características nutricionales de medio de cultivo?*

#### Objetivos:

**General:** Utilizar un algoritmo computacional para determinar la unión de Lrp a distintos sitios del genoma durante las fases de crecimiento de _E. coli_ en respuesta a tres condiciones nutricionales distintas.

**Específico 1:** Manipular las bases de datos obtenidas en esta fase del experimento a través de la utilización de comandos de Unix y R para crear directorios, archivos y tablas útiles que permitan responder la pregunta de investigación.

**Específico 2:** Crear un reporte que incluya la secuencia lógica del algoritmo utilizado, así como descripciones gráficas (a través de RStudio) de los resultados obtenidos.

#### Metodología:

Se utilizaron datos de RegulonDB obtenidos de experimentos realizados en _E. coli_, donde las bacterias fueron expuestas a tres distintas condiciones nutricionales: medio mínimo, medio mínimo + aminoácidos ramificados (LIV) o medio enriquecido, y estudiadas durante tres fases de crecimiento: logarítmica, de transición y estacionaria. Esto resultó en nueve condiciones distintas. Se archivaron los datasets en carpetas creadas para este proyecto, y se procesaron los archivos a través de comandos en la terminal de Unix, así como en RStudio, para identificar los sitios de unión únicos reportados en las nueve condiciones, así como para evaluar las diferencias en la unión de estos sitios a Lrp de acuerdo con las condiciones nutricionales y la fase de crecimiento.

#### Resultados:

Se identificaron 34 sitios de unión en las nueve condiciones del experimento. La unión de 28 de estos sitios (82%) a Lrp cambiaba de acuerdo con la fase de crecimiento y/o las condiciones nutricionales. La fase estacionaria de bacterias en medio mínimo alcanzó el máximo número de sitos unidos a Lrp, comparada con el resto de condiciones. Hubo un incremento en la variabilidad de cada sitio de unión a Lrp atribuida a los nutrientes del medio de acuerdo con la fase de crecimiento (log<trans<stat), alcanzando unión diferencial a Lrp en 23 sitios durante la fase estacionaria.

#### Conclusión:

La unión de Lrp al genoma de _E. coli_ se ve influenciada por las características nutricionales del medio, así como por la fase de crecimiento en el que se encuentran las bacterias.

---

# Algoritmo de comandos

## Obtención de datos y generación de carpetas

Para realizar este proyecto, se solicitaron datos al profesor (GSE111874.zip), los cuales fueron enviados por correo electrónico y descargados a la carpeta local. Estos fueron obtenidos de experimentos depositados en RegulonDB

Localización del archivo:

```bash
MacBook-Air-de-Miguel:Downloads mikemtzrojas$ pwd
/Users/mikemtzrojas/Downloads
MacBook-Air-de-Miguel:Downloads mikemtzrojas$ ls
GSE111874.zip
```

Para empezar a trabajar, primero se crearon carpetas donde se guardarán los datos y donde se almacenará la información generada.

```bash
cd /Users/mikemtzrojas/Desktop/
mkdir ./Proyecto_final_2021
cd Proyecto_final_2021
mkdir data results scripts
ls
data	results	scripts
```

Transladamos el archivo comprimido GSE111874.zip a su carpeta correspondiente

```bash
mv /Users/mikemtzrojas/Downloads/GSE111874.zip  ./data/
ls ./data/
GSE111874.zip
```

Finalmente, guardo este reporte como nuevo archivo de markdown en la carpeta correspondiente (scripts)

```bash
ls ./scripts/
Reporte_final_markdown.md
```

---

## Exploración inicial de los datos e identificación del problema

Primero descomprimimos el archivo enviado, obteniendo dos carpetas

```bash
unzip GSE111874.zip
ls
GSE111874	GSE111874.zip	__MACOSX
```

La carpeta GSE111874 contiene lo siguiente: 9 carpetas que contienen los archivos de secuenciación, una tabla de resumen de datos separada por comas y el mismo archivo de resumen en Numbers.

```bash
cd GSE111874
ls -F
DS00069/			DS00073/			DS00077/
DS00070/			DS00074/			SummaryDatasets.csv
DS00071/			DS00075/			SummaryDatasets.numbers*
DS00072/			DS00076/
```

Decidimos explorar el resumen separado por comas, el cual contiene 10 filas, la primera correspondiente a encabezados, 11 columnas especificadas y 3 sin nombre.

```bash
wc -l SummaryDatasets.csv
10
head -n1 SummaryDatasets.csv | perl -ne 's/,/\n/g; print;'
dataset_id
pmid
author
series_id
strategy
layout
test
control
protein_name
peak_file
site_file



head -n1 SummaryDatasets.csv | perl -ne 's/,/\n/g; print;' | wc -l
      14
```

En resumen, la disposición de columnas es:

1. Identificador del dataset (DS00069-77)
2. Identificador de Pubmed (30420454)
3. Autor (Kroner2019 (Freddolino) en todos los casos)
4. Identificador de serie (GSE111874 en todos los casos)
5. Estrategia experimental (ChIP-seq)
6. Layout (pareado)
7. Test (GSM3043051-68)
8. Control (GSM3043087-3104)
9. Proteina (Lrp)
10. Archivo de Picos de lectura (ChIP-seq_collection/GSE111874/datasets/DS000XX/DS000XX_peaks.bed, donde XX= 69-77)
11. Archivo de sitios (ChIP-seq_collection/GSE111874/datasets/DS000XX/DS000XX_sites.bed, donde XX= 69-77)
12. Descripción extensa de condiciones en el crecimiento (Cepa, genotipo, medio, temperatura, fase de crecimiento, agitación)
13. Nueve Condiciones relevantes (Medio= Mínimo + LIV, Enriquecido, Mínimo; fase= Logarítmica, Transición, Estacionaria)
14. Origen (?) (2-ChIPseq extracted)

**Con esta información, surgen las siguientes preguntas:**

- ¿A qué sitios se une la proteína Lrp en E. coli?

- ¿Estos sitios cambian durante la fase de crecimiento de la bacteria?

- ¿Esta regulación se modifica en distintas condiciones nutricionales (medio)?

Para abordar estas preguntas, requerimos extraer la información de las lecturas, las cuales se encuentras en carpetas categorizadas por dataset (DS00069-77). Cada dataset consta de los datos procesados y estandarizados de cada experimento, organizados en cinco archivos que muestran las regiones del genoma con picos de lectura significativos para la unión al factor de transcripción, así como los motivos de unión identificados en el experimento.

Ejemplo de archivos por dataset (5 archivos)

```bash
ls -F ./DS00069
DS00069_peaks.bed	DS00069_peaks.fasta	DS00069_peaks.ft	DS00069_sites.bed	DS00069_sites.fasta
```

Breve descripción del contenido de cada archivo:

1. *peaks.bed: Tablas que indican los picos de lectura, el locus, la posición limitada, el enriquecimiento (FC) y log10 del valor p para el sitio "cumbre" de unión estimado en cada pico.
2. *peaks.fasta: Contiene la secuenciación de los sitios especificados en (1). Se organiza en pares de líneas, cada par indicando el pico de lectura y su secuencia correspondiente.
3. *peaks.ft: Matriz de "position weight" que contiene el análisis del archivo de secuenciación *peaks.fasta, utilizando la referencia *.freq para generar la referencia relativa (lambda control), y arroja un reporte de significancia en los sitios unidos a Lrp vs Background.
4. *sites.bed: Tablas que indican los motivos de unión identificados, el pico de lectura correspondiente, coordenadas en pares de bases, hebra y secuencia específica. Los datos corresponden con el reporte de (3).
5. *sites.fasta: Con estructura similar a (2), contiene los sitios de unión especificados en (4) y sus secuencias.

Se decide explorar el número de sitios de unión para Lrp arrojados en cada dataset, lo cual correspondería a las nueve diferentes condiciones experimentales indicadas en el Resumen, basándonos en los archivos *sites.bed

```bash
MacBook-Air-de-Miguel:GSE111874 mikemtzrojas$ grep -v '^#' ./DS00069/DS00069_sites.bed | wc -l
       6
MacBook-Air-de-Miguel:GSE111874 mikemtzrojas$ grep -v '^#' ./DS00070/DS00070_sites.bed | wc -l
      21
MacBook-Air-de-Miguel:GSE111874 mikemtzrojas$ grep -v '^#' ./DS00071/DS00071_sites.bed | wc -l
      24
MacBook-Air-de-Miguel:GSE111874 mikemtzrojas$ grep -v '^#' ./DS00072/DS00072_sites.bed | wc -l
       8
MacBook-Air-de-Miguel:GSE111874 mikemtzrojas$ grep -v '^#' ./DS00073/DS00073_sites.bed | wc -l
      12
MacBook-Air-de-Miguel:GSE111874 mikemtzrojas$ grep -v '^#' ./DS00074/DS00074_sites.bed | wc -l
       8
MacBook-Air-de-Miguel:GSE111874 mikemtzrojas$ grep -v '^#' ./DS00075/DS00075_sites.bed | wc -l
      13
MacBook-Air-de-Miguel:GSE111874 mikemtzrojas$ grep -v '^#' ./DS00076/DS00076_sites.bed | wc -l
      19
MacBook-Air-de-Miguel:GSE111874 mikemtzrojas$ grep -v '^#' ./DS00077/DS00077_sites.bed | wc -l
      25
```

Al revisar los datos contenidos en los archivos *sites.bed encuentro que se reportan algunos sitios comunes entre condiciones (mismo sitio de inicio y término), por lo que creo archivos que contengan las columnas 2,3,6 y 7 de cada archivo *.sites.bed (start, stop, strand, sequence), las copio y las guardo en archivos de texto utilizando nano con el nombre pXX.txt donde XX=69-77. Finalmente concateno los 9 archivos y los inserto en un nuevo archivo llamado summ_sites.txt. Los archivos de txt generados se pasaron a la carpeta "results". 

```bash
cut -f2,3,6,7 ./DS00069/DS00069_sites.bed
nano
...
cat p69 p70 p71 p72 p73 p74 p75 p76 p77
nano

cd ../..
mkdir ./results/peaks_per_dataset

mv ./data/GSE111874/p77.txt ./results/peaks_per_dataset
ls ./results/peaks_per_dataset
p69.txt	p70.txt	p71.txt	p72.txt	p73.txt	p74.txt	p75.txt	p76.txt	p77.txt

mv ./data/GSE111874/summ_sites.txt ./results/
ls ./results/
peaks_per_dataset	summ_sites.txt

```

Después, encuentro que hay 136 sitios en total reportados en las distintas condiciones (una línea corresponde al encabezado). Ejecuto un comando para ordenar los sitios de inicio de todas las condiciones, elimino repeticiones y cuento los sitios de unión únicos, encontrando 34 sitios distintos en todas las condiciones (una línea corresponde al encabezado). Llama la atención que 6 de ellos se mantienen en todas las condiciones y 9 solo en una.

```bash
tail -n +2 summ_sites.txt | wc -l
     136
cut -f1,2 summ_sites.txt | tail -n +2 | sort -n | uniq | wc -l
      34
cut -f1,2 summ_sites.txt | tail -n +2 | sort -n | uniq -c | sort -r
   9 85373	85386
   9 419323	419336
   9 3354285	3354298
   9 3050864	3050877
   9 1237528	1237541
   9 1237383	1237396
   8 3485372	3485385
   7 84171	84184
   7 2949193	2949206
   5 3343096	3343109
   5 2939499	2939512
   4 3709631	3709644
   4 3258198	3258211
   4 2241043	2241056
   4 1297260	1297273
   3 605760	605773
   3 4634587	4634600
   3 4269956	4269969
   3 2037674	2037687
   3 193488	193501
   2 846510	846523
   2 754885	754898
   2 590710	590723
   2 3653695	3653708
   2 3375206	3375219
   1 662259	662272
   1 4081494	4081507
   1 3595243	3595256
   1 344943	344956
   1 2922241	2922254
   1 2629602	2629615
   1 1990138	1990151
   1 1958418	1958431
   1 1772701	1772714
```

Se creó un archivo .txt conteniendo los sitios de unión a Lrp únicos identificados en las distintas condiciones. El archivo se nombró como ordered_sites.txt, también se guardó en la carpeta "results".

```bash
tail -n +2 ./results/summ_sites.txt | sort -n | uniq | sort -n
nano
pwd
/Users/mikemtzrojas/Desktop/Proyecto_final_2021/results
ls -F
peaks_per_dataset/ summ_sites.txt ordered_sites.txt
```

Se decide organizar los datos en una tabla que muestre los diferentes sitios de unión reportados de acuerdo con las distintas condiciones experimentales. Para esto se continúa el proyecto utilizando R.
