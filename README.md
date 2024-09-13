# Pipeline Metabarcoding

## Carpeta de trabajo de proyecto

`/media/prosopis/E/LGBB/Server/user/fast5_files`

> **Comentario:** Es útil especificar la ruta del proyecto para mantener la organización.

## Obtención del programa dorado

Página web: https://github.com/nanoporetech/dorado

> **Comentario:** Dorado es un programa de basecalling y análisis de datos de secuenciación de nanoporos desarrollado por Oxford Nanopore Technologies.

```bash
pwd # Te mostrará la siguiente ruta /home/user/
mkdir Bioprograms
cd Bioprograms
wget https://cdn.oxfordnanoportal.com/software/analysis/dorado-0.7.2-linux-x64.tar.gz
```

> **Comentario:** Estas líneas de código descargan el programa dorado desde el sitio web de Oxford Nanopore.

## Descomprimir el programa

```bash
tar xvfz dorado-0.7.2-linux-x64.tar.gz
cd dorado-0.7.2-linux-x64
cd bin
pwd # /home/user/Bioprograms/dorado-0.7.2-linux-x64/bin
sudo ln -s /home/user/Bioprograms/dorado-0.7.2-linux-x64/bin/dorado /usr/local/bin/dorado
dorado --help
```

> **Comentario:** Estas líneas de código descomprimen el archivo descargado y crean un enlace simbólico al ejecutable de dorado en `/usr/local/bin/`, lo que permite ejecutar dorado desde cualquier lugar en el sistema.

## Instalar POD5

Convertir fast5 a pod5 antes del basecalling

> **Comentario:** POD5 es un formato de archivo desarrollado por Oxford Nanopore Technologies para almacenar datos de secuenciación de nanoporos. Es un formato más eficiente y comprimido que FAST5.

```bash
pip install pod5 (nivel de usuario)
```

> **Comentario:** Esta línea de código instala la biblioteca de Python `pod5`, que se utiliza para interactuar con archivos POD5.

## Ejecutar POD5

```bash
pod5 convert fast5 fast5/*.fast5 --output converted.pod5
```

> **Comentario:** Esta línea de código utiliza `pod5` para convertir todos los archivos FAST5 en el directorio `fast5` a un único archivo POD5 llamado `converted.pod5`.

## Descargar modelos con dorado

```bash
mkdir dorado_models
cd dorado_models
dorado download
mkdir sup
mv *_sup@* sup
mkdir fast
mv *_fast@* fast
mkdir hac
mv *_hac@* hac
mkdir stereo
mv *_stereo@* stereo/
ln -s /media/prosopis/E/LGBB/Server/rodrigo.suarez/Endofitos_ramas/dorado_models/sup /media/prosopis/E/LGBB/Server/rodrigo.suarez/Endofitos_ramas/all_fast5/sup
```

> **Comentario:** Estas líneas de código descargan los modelos basecaller de dorado y los organizan en directorios.

## Realizamos el llamado de las bases

```bash
dorado basecaller sup --kit-name SQK-16S024 --min-qscore 10 --barcode-both-ends converted.pod5 > calls.bam
dorado demux --kit-name SQK-16S024 --output-dir barcodes --emit-summary calls.bam
cd barcodes
```

> **Comentario:** 
> - `--kit-name SQK-16S024`: Especifica el nombre del kit de secuenciación utilizado.
> - `--min-qscore 10`: Establece un umbral de calidad mínimo de 10 para las lecturas.
> - `--barcode-both-ends`: Indica que los códigos de barras están presentes en ambos extremos de las lecturas.
> - `> calls.bam`: Redirige la salida a un archivo BAM llamado `calls.bam`.
> - `--output-dir barcodes`: Especifica el directorio de salida para los archivos demultiplexados.
> - `--emit-summary`: Genera un resumen de la ejecución de secuenciación.

## Resumen de la secuenciación 

```bash
for file in *.bam; do prefix="${file%.bam}"; dorado summary "$file" > "${prefix}_summary.tsv"; done
```

> **Comentario:** Genera un resumen para cada archivo BAM en el directorio y lo guarda en un archivo TSV.

## Convertimos de bam a fastq

```bash
for file in *.bam; do prefix="${file%.bam}"; samtools sort -n "$file" -o "${prefix}_sorted.bam"; done
for file in *_sorted.bam; do prefix="${file%.bam}"; bedtools bamtofastq -i "${prefix}.bam" -fq "${prefix}.fastq"; done
seqkit stats -a -j 4 *_sorted.fastq > stats_sorted_fastq.txt
```

> **Comentario:** 
> - `samtools sort -n`: Ordena los archivos BAM por nombre de lectura.
> - `bedtools bamtofastq`: Convierte los archivos BAM ordenados a formato FASTQ.

```
file                               format  type  num_seqs      sum_len  min_len  avg_len  max_len       Q1       Q2     Q3  sum_gap    N50  Q20(%)  Q30(%)
SQK-16S024_barcode01_sorted.fastq  FASTQ   DNA      2,232    2,812,246        2    1,260    3,808    1,230    1,405  1,455        0  1,421   50.76    8.48
SQK-16S024_barcode02_sorted.fastq  FASTQ   DNA      1,626    1,903,444        4  1,170.6    1,953      980    1,387  1,452        0  1,413   50.53    8.58
SQK-16S024_barcode03_sorted.fastq  FASTQ   DNA     12,802   15,342,892        7  1,198.5    2,877    1,077    1,394  1,445        0  1,408   50.64    7.79
SQK-16S024_barcode04_sorted.fastq  FASTQ   DNA      8,550    9,958,508        9  1,164.7    2,091    1,029    1,387  1,436        0  1,402    50.3     7.5
SQK-16S024_barcode05_sorted.fastq  FASTQ   DNA        678      871,012       59  1,284.7    1,937    1,333    1,404  1,455        0  1,419   49.97    7.33
SQK-16S024_barcode06_sorted.fastq  FASTQ   DNA      3,192    4,003,420        3  1,254.2    2,250    1,306    1,407  1,452        0  1,421   51.25    7.43
SQK-16S024_barcode07_sorted.fastq  FASTQ   DNA     12,476   14,802,626        4  1,186.5    2,838    1,053    1,395  1,448        0  1,409   50.54    7.45
SQK-16S024_barcode08_sorted.fastq  FASTQ   DNA        312      437,058       28  1,400.8    1,861  1,389.5    1,421  1,454        0  1,424   50.44    8.41
SQK-16S024_barcode09_sorted.fastq  FASTQ   DNA     22,938   18,677,366        2    814.3    1,993      346      873  1,335        0  1,254   51.16    7.87
SQK-16S024_barcode10_sorted.fastq  FASTQ   DNA     17,742   19,764,174        8    1,114    2,654      717    1,308  1,409        0  1,391   50.81    8.19
SQK-16S024_barcode11_sorted.fastq  FASTQ   DNA     13,662   14,534,452        1  1,063.9    2,029      838    1,192  1,384        0  1,337   50.87    7.85
SQK-16S024_barcode12_sorted.fastq  FASTQ   DNA     11,866   13,408,054       10    1,130    2,012      944    1,335  1,399        0  1,377   50.37     7.7
SQK-16S024_barcode13_sorted.fastq  FASTQ   DNA     13,526   15,222,272        1  1,125.4    2,696      914    1,326  1,398        0  1,374   50.65    7.65
SQK-16S024_barcode14_sorted.fastq  FASTQ   DNA     16,278   15,979,112        2    981.6    2,320      504    1,126  1,392        0  1,359   50.59    8.19
SQK-16S024_barcode15_sorted.fastq  FASTQ   DNA     14,378   16,015,288        2  1,113.9    2,440      831    1,338  1,402        0  1,388   50.57    7.33
SQK-16S024_barcode16_sorted.fastq  FASTQ   DNA     14,146   15,581,836        2  1,101.5    2,109      814    1,300  1,400        0  1,379   50.65     7.9
SQK-16S024_barcode17_sorted.fastq  FASTQ   DNA     17,988   18,836,740        2  1,047.2    3,057      624  1,304.5  1,399        0  1,380   50.24    7.07
SQK-16S024_barcode18_sorted.fastq  FASTQ   DNA     16,030   16,605,426        2  1,035.9    2,088      537    1,287  1,399        0  1,386   50.48    7.85
unclassified_sorted.fastq          FASTQ   DNA    474,168  648,009,092        7  1,366.6    4,216    1,393    1,405  1,421        0  1,406   53.45    8.42
```

## Remoción de Adaptadores y Control de Calidad

Usemos Porechop (v0.2.4) para la remoción de adaptadores.

> **Comentario:** Porechop es una herramienta para recortar adaptadores y secuencias de cebadores de datos de secuenciación de nanoporos.

```bash
ml porechop
for file in *.fastq; do prefix="${file%_sorted.fastq}"; porechop-runner.py -i "${prefix}_sorted.fastq" -o "${prefix}_trimmed.fastq"; done
```

> **Comentario:** 
> - `-i`: Especifica el archivo de entrada.
> - `-o`: Especifica el archivo de salida después de la remoción de adaptadores.

Emplearemos Chopper (v0.8.0) en lugar de Nanofilt para filtrar las secuencias por calidad y longitud

> **Comentario:** Chopper es una herramienta para filtrar lecturas de secuenciación de nanoporos por calidad y longitud.

```bash
ml chopper
for file in *_trimmed.fastq; do prefix="${file%_trimmed.fastq}"; chopper -i "${prefix}_trimmed.fastq" -c ppa_v2.asm.fasta  --quality 10 --minlength 1000; done
```

> **Comentario:** 
> - `-i`: Especifica el archivo de entrada.
> - `-c`: Especifica el archivo de referencia para la corrección.
> - `--quality 10`: Filtra las lecturas con una calidad mínima de 10.
> - `--minlength 1000`: Filtra las lecturas con una longitud mínima de 1000 bases.

## Estadísticas de Secuencias

Generaremos las estadisticas de las secencias con Nanoplot (v1.42.0)

> **Comentario:** NanoPlot es una herramienta para generar gráficos de resumen de datos de secuenciación de nanoporos.

```bash
conda install -c bioconda nanoplot (Por usuario)
nanoplot --fastq filtered.fastq --loglength
for file in *.fastq; do prefix="${file%.fastq}"; NanoPlot --fastq "${prefix}.fastq" --loglength; done
```

> **Comentario:** 
> - `--fastq`: Especifica el archivo FASTQ de entrada.
> - `--loglength`: Genera gráficos con el logaritmo de la longitud de las lecturas.

## Descarga, instalación y Ejecución de NanoCLUST

> **Comentario:** NanoCLUST es una herramienta para la agrupación y clasificación taxonómica de lecturas de secuenciación de amplicones 16S rRNA.

- Instalación de requerimientos:

```bash
conda install -c conda-forge graphviz
```

```bash
git clone https://github.com/genomicsITER/NanoCLUST.git
cd NanoCLUST
```

> **Comentario:** Estas líneas de código instalan `graphviz` y clonan el repositorio de NanoCLUST desde GitHub.

# Primero necesitamos descargar la base de datos del NCBI

```bash
$ mkdir db db/taxdb
$ wget https://ftp.ncbi.nlm.nih.gov/blast/db/16S_ribosomal_RNA.tar.gz && tar -xzvf 16S_ribosomal_RNA.tar.gz -C db
$ wget https://ftp.ncbi.nlm.nih.gov/blast/db/taxdb.tar.gz && tar -xzvf taxdb.tar.gz -C db/taxdb
```

> **Comentario:** Estas líneas de código descargan la base de datos del NCBI 16S ribosomal RNA y la base de datos de taxonomía.

# Ejecutamos el test para corroborrar la instalación de los programas

```bash
$ cd NanoCLUST/
$ ml nextflow
$ nextflow run main.nf -profile test,docker
```

> **Comentario:** 
> - `-profile test,docker`: Ejecuta el flujo de trabajo en modo de prueba utilizando Docker para la containerización.

# Para ejecutar las muestras

```bash
cd trimmed
$ for file in *.fastq; do prefix="${file%.fastq}"; nextflow run ../NanoCLUST/main.nf -profile docker --reads "${prefix}.fastq" --db ../NanoCLUST/db/16S_ribosomal_RNA --tax ../NanoCLUST/db/taxdb/ --outdir ../NanoCLUST/"${prefix}_ouput" --multiqc; done
```

> **Comentario:** 
> - `--reads`: Especifica el archivo de lecturas FASTQ de entrada.
> - `--db`: Especifica la base de datos de 16S ribosomal RNA.
> - `--tax`: Especifica la base de datos de taxonomía.
> - `--outdir`: Especifica el directorio de salida para los resultados.
> - `--multiqc`: Genera un informe de calidad múltiple con MultiQC.
