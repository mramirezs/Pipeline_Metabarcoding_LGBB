# Pipeline Metabarcoding

## Carpeta de trabajo de proyecto

/media/prosopis/E/LGBB/Server/Virome_project_10_07_24/VIRALG/20231116_2135_MC-112831_AQZ121_7223a9ab

## Obtención del programa dorado

Página web: https://github.com/nanoporetech/dorado

```bash
pwd # /home/user/
mkdir Bioprograms
cd Bioprograms
wget https://cdn.oxfordnanoportal.com/software/analysis/dorado-0.7.2-linux-x64.tar.gz
```

## Descomprimir el programa

```bash
tar xvfz dorado-0.7.2-linux-x64.tar.gz
cd dorado-0.7.2-linux-x64
cd bin
pwd # /media/prosopis/E/LGBB/Server/Bioprograms/dorado-0.7.2-linux-x64/bin
sudo ln -s /media/prosopis/E/LGBB/Server/Bioprograms/dorado-0.7.2-linux-x64/bin/dorado /usr/local/bin/dorado (escribirlo)
dorado --help
```

## Instalar POD5

Convertir fast5 a pod5 antes del basecalling

```bash
pip install pod5 (A nivel de usuario)
```

## Ejecutar POD5

```bash
pod5 convert fast5 fast5/*.fast5 --output converted.pod5

```

## Descargar modelos con dorado

```bash
dorado download
```

## Realizamos el llamado de las bases

```
dorado basecaller sup converted.pod5 > calls.bam

## Realizamos el trimado de las secuencias

```bash
dorado trim calls.bam > trimmed.bam
```

## Obtuvimos el resumen de la secuenciación 

```bash
dorado summary trimmed.bam > summary.tsv 
```
## Convertimos de bam a fastq

```bash
samtools sort -n trimmed.bam -o trimmed_sorted.bam
bedtools bamtofastq -i trimmed_sorted.bam -fq trimmed_sorted.fastq
```

## Remoción de Adaptadores y Control de Calidad

Usemos Porechop (v0.2.4) para la remoción de adaptadores.

```bash
ml porechop
porechop-runner.py -i basecalled/ -o trimmed/
```

Emplearemos Chopper (v0.8.0) en lugar de Nanofilt para filtrar las secuencias por calidad y longitud

```bash
ml chopper
chopper -i trimmed/*.fastq -o filtered.fastq --quality-threshold 10 --min-length 1000
```

## Estadísticas de Secuencias

Generaremos las estadisticas de las secencias con Nanoplot (v1.42.0)

```bash
conda install -c bioconda nanoplot (Por usuario)
nanoplot --fastq filtered.fastq --loglength
```


## Ejecución de NanoCLUST

```bash
conda install -c conda-forge graphviz
```



```bash
# Descarga del programa
git clone https://github.com/genomicsITER/NanoCLUST.git
cd NanoCLUST


# Primero necesitamos descargar la base de datos del NCBI
$ mkdir db db/taxdb
$ wget https://ftp.ncbi.nlm.nih.gov/blast/db/16S_ribosomal_RNA.tar.gz && tar -xzvf 16S_ribosomal_RNA.tar.gz -C db
$ wget https://ftp.ncbi.nlm.nih.gov/blast/db/taxdb.tar.gz && tar -xzvf taxdb.tar.gz -C db/taxdb

# Ejecutamos el test para corroborrar la instalación de los programas
$ cd NanoCLUST/
$ ls
$ ml nextflow
$ nextflow run main.nf -profile test,docker



# Para ejecutar las muestras

$ nextflow run main.nf -profile docker --reads filtered_porechop_48SE_1.fastq --db db/16S_ribosomal_RNA --tax db/taxdb/ --outdir 48SE --multiqc
```
