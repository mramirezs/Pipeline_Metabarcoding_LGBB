# Pipeline Metabarcoding

## Carpeta de trabajo de proyecto

/media/prosopis/E/LGBB/Server/user/fast5_files

## Obtención del programa dorado

Página web: https://github.com/nanoporetech/dorado

```bash
pwd # Te mostrará la siguiente ruta /home/user/
mkdir Bioprograms
cd Bioprograms
wget https://cdn.oxfordnanoportal.com/software/analysis/dorado-0.7.2-linux-x64.tar.gz
```

## Descomprimir el programa

```bash
tar xvfz dorado-0.7.2-linux-x64.tar.gz
cd dorado-0.7.2-linux-x64
cd bin
pwd # /home/user/Bioprograms/dorado-0.7.2-linux-x64/bin
sudo ln -s /home/user/Bioprograms/dorado-0.7.2-linux-x64/bin/dorado /usr/local/bin/dorado
dorado --help
```

## Instalar POD5

Convertir fast5 a pod5 antes del basecalling

```bash
pip install pod5 (nivel de usuario)
```

## Ejecutar POD5

```bash
pod5 convert fast5 fast5/*.fast5 --output converted.pod5
```

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

## Realizamos el llamado de las bases

```bash
dorado basecaller sup --kit-name SQK-16S024 --min-qscore 10 --barcode-both-ends converted.pod5 > calls.bam
dorado demux --kit-name SQK-16S024 --output-dir barcodes --emit-summary calls.bam
cd barcodes
```

## Resumen de la secuenciación 

```bash
for file in *.bam; do prefix="${file%.bam}"; dorado summary "$file" > "${prefix}_summary.tsv"; done
```
## Convertimos de bam a fastq

```bash
for file in *.bam; do prefix="${file%.bam}"; samtools sort -n "$file" -o "${prefix}_sorted.bam"; done
for file in *_sorted.bam; do prefix="${file%.bam}"; bedtools bamtofastq -i "${prefix}.bam" -fq "${prefix}.fastq"; done
```

## Remoción de Adaptadores y Control de Calidad

Usemos Porechop (v0.2.4) para la remoción de adaptadores.

```bash
ml porechop
for file in *.fastq; do prefix="${file%_sorted.fastq}"; porechop-runner.py -i "${prefix}_sorted.fastq" -o "${prefix}_trimmed.fastq"; done
```

Emplearemos Chopper (v0.8.0) en lugar de Nanofilt para filtrar las secuencias por calidad y longitud

```bash
ml chopper
for file in *.fastq; do prefix="${file%_trimmed.fastq}"; chopper -i "${prefix}_trimmed.fastq" -c ppa_v2.asm.fasta  --quality 10 --minlength 1000; done
```

## Estadísticas de Secuencias

Generaremos las estadisticas de las secencias con Nanoplot (v1.42.0)

```bash
conda install -c bioconda nanoplot (Por usuario)
nanoplot --fastq filtered.fastq --loglength
for file in *.fastq; do prefix="${file%.fastq}"; NanoPlot --fastq "${prefix}.fastq" --loglength; done
```

## Descarga, instalación y Ejecución de NanoCLUST

- Instalación de requerimientos:

```bash
conda install -c conda-forge graphviz
```

```bash
git clone https://github.com/genomicsITER/NanoCLUST.git
cd NanoCLUST
```

# Primero necesitamos descargar la base de datos del NCBI

```bash
$ mkdir db db/taxdb
$ wget https://ftp.ncbi.nlm.nih.gov/blast/db/16S_ribosomal_RNA.tar.gz && tar -xzvf 16S_ribosomal_RNA.tar.gz -C db
$ wget https://ftp.ncbi.nlm.nih.gov/blast/db/taxdb.tar.gz && tar -xzvf taxdb.tar.gz -C db/taxdb
```

# Ejecutamos el test para corroborrar la instalación de los programas

```bash
$ cd NanoCLUST/
$ ml nextflow
$ nextflow run main.nf -profile test,docker
```

# Para ejecutar las muestras

```bash
cd trimmed
$ for file in *.fastq; do prefix="${file%.fastq}"; nextflow run ../NanoCLUST/main.nf -profile docker --reads "${prefix}.fastq" --db ../NanoCLUST/db/16S_ribosomal_RNA --tax ../NanoCLUST/db/taxdb/ --outdir ../NanoCLUST/"${prefix}_ouput" --multiqc; done
```
