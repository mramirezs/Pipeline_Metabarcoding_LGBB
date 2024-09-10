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
```

> **Comentario:** 
> - `samtools sort -n`: Ordena los archivos BAM por nombre de lectura.
> - `bedtools bamtofastq`: Convierte los archivos BAM ordenados a formato FASTQ.

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
for file in *.fastq; do prefix="${file%_trimmed.fastq}"; chopper -i "${prefix}_trimmed.fastq" -c ppa_v2.asm.fasta  --quality 10 --minlength 1000; done
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
> - `-profile test,docker`: Ejecuta el flujo de trabajo en modo de prueba utilizando Docker para la contenedorización.

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
