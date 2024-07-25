# Pipeline_Metabarcoding

## Carpeta de trabajo de proyecto

/media/prosopis/E/LGBB/Server/Virome_project_10_07_24/VIRALG/20231116_2135_MC-112831_AQZ121_7223a9ab

## Obtención del programa dorado

Página web: https://github.com/nanoporetech/dorado

```
pwd # /home/user/
mkdir Bioprograms
cd Bioprograms
wget https://cdn.oxfordnanoportal.com/software/analysis/dorado-0.7.2-linux-x64.tar.gz
```

## Descomprimir el programa

```
tar xvfz dorado-0.7.2-linux-x64.tar.gz
cd dorado-0.7.2-linux-x64
cd bin
pwd # /media/prosopis/E/LGBB/Server/Bioprograms/dorado-0.7.2-linux-x64/bin
sudo ln -s /media/prosopis/E/LGBB/Server/Bioprograms/dorado-0.7.2-linux-x64/bin/dorado /usr/local/bin/dorado (escribirlo)
dorado --help
```

## Instalar POD5

Convertir fast5 a pod5 antes del basecalling

```
pip install pod5 (A nivel de usuario)
```

```
sudo pip install virtualenv 
sudo mkdir -p /opt/virtualenvs
sudo virtualenv /opt/virtualenvs/globalenv
sudo /opt/virtualenvs/globalenv/bin/pip install pod5 

# Cualquier usuario puede activar el entorno virtual para usar pod5:
source /opt/virtualenvs/globalenv/bin/activate
```

## Ejecutar POD5

```
pod5 convert fast5 fast5_pass/*.fast5 --output converted_pass.pod5

pod5 convert fast5 fast5_fail/*.fast5 --output converted_fail.pod5
```

## Descargar modelos con dorado

```
dorado download
```

## Realizamos el llamado de las bases

```
dorado basecaller hac converted.pod5 > calls.bam
```

```
dorado basecaller sup converted_pass.pod5 > calls_pass.bam

# Results
[2024-07-16 12:37:31.269] [info] Running: "basecaller" "sup" "converted_pass.pod5"
[2024-07-16 12:37:31.364] [info] > Creating basecall pipeline
[2024-07-16 12:39:01.023] [info] cuda:0 using chunk size 10000, batch size 512
[2024-07-16 12:39:15.345] [info] cuda:0 using chunk size 5000, batch size 768
[2024-07-16 12:49:47.604] [info] > Simplex reads basecalled: 30974
[2024-07-16 12:49:47.604] [info] > Basecalled @ Samples/s: 2.369899e+05
[2024-07-16 12:49:47.613] [info] > Finished
```

```
dorado basecaller sup converted_fail.pod5 > calls_fail.bam
# Results
[2024-07-16 12:53:36.714] [info] Running: "basecaller" "sup" "converted_fail.pod5"
[2024-07-16 12:53:36.725] [info] > Creating basecall pipeline
[2024-07-16 12:55:05.368] [info] cuda:0 using chunk size 10000, batch size 512
[2024-07-16 12:55:19.663] [info] cuda:0 using chunk size 5000, batch size 768
[2024-07-16 13:00:42.777] [info] > Simplex reads basecalled: 11154
[2024-07-16 13:00:42.777] [info] > Simplex reads filtered: 5
[2024-07-16 13:00:42.777] [info] > Basecalled @ Samples/s: 2.548511e+05
[2024-07-16 13:00:42.782] [info] > Finished
```

## Realizamos el trimado de las secuencias

```
dorado trim calls_fail.bam > trimmed_fail.bam

dorado trim calls_pass.bam > trimmed_pass.bam
```

## Obtuvimos el resumen de la secuenciación 

```
dorado summary trimmed_fail.bam > summary_fail.tsv 

dorado summary trimmed_pass.bam > summary_pass.tsv 
```

## Convertimos de bam a fastq

```
samtools sort -n trimmed_fail.bam -o trimmed_fail_sorted.bam
bedtools bamtofastq -i trimmed_fail_sorted.bam -fq trimmed_fail_sorted.fastq
```

```
samtools sort -n trimmed_pass.bam -o trimmed_pass_sorted.bam
bedtools bamtofastq -i trimmed_pass_sorted.bam -fq trimmed_pass_sorted.fastq
```
# Visualizar calidad de las lecturas

```
fastqc trimmed_fail_sorted.fastq
```

```
fastqc trimmed_pass_sorted.fastq
```

