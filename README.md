# Pipeline_Metabarcoding

> Laboratorio de Genómica y Bioinformática de la Bioindiversidad

## Carpeta de trabajo de proyecto

Verificar la ruta absoluto de cada usuario:

```
/media/prosopis/E/LGBB/Server/Usuario
```

## Pre-Procesamiento

### POD5 v.0.3.12

Convertir fast5 a pod5 antes del basecalling

#### Instalación

Para usuarios.

```
pip install pod5
```

#### Ejecución

Para lecturas fast5 que son pass.
```
pod5 convert fast5 fast5_pass/*.fast5 --output converted_pass.pod5
```

Para lecturas fast5 que son pass.
```
pod5 convert fast5 fast5_fail/*.fast5 --output converted_fail.pod5
```

### Dorado v.0.7.2

#### Obtención

```
pwd # /home/user/
mkdir Bioprograms
cd Bioprograms
wget https://cdn.oxfordnanoportal.com/software/analysis/dorado-0.7.2-linux-x64.tar.gz
```

#### Descomprimir

Solo para superusuario.

```
tar xvfz dorado-0.7.2-linux-x64.tar.gz
cd dorado-0.7.2-linux-x64
cd bin
pwd # /media/prosopis/E/LGBB/Server/Bioprograms/dorado-0.7.2-linux-x64/bin
sudo ln -s /media/prosopis/E/LGBB/Server/Bioprograms/dorado-0.7.2-linux-x64/bin/dorado /usr/local/bin/dorado (escribirlo)
dorado --help
```

#### Descarga de modelos 

```
ml dorado (A nivel de usuario)
dorado download
```

#### Ejecución

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

#### Trimado
```
dorado trim calls_fail.bam > trimmed_fail.bam

dorado trim calls_pass.bam > trimmed_pass.bam
```

#### Obtención de estadísticas

```
dorado summary trimmed_fail.bam > summary_fail.tsv 
```

```
dorado summary trimmed_pass.bam > summary_pass.tsv 
```

#### Convertimos de bam a fastq

```
samtools sort -n trimmed_pass.bam -o trimmed_pass_sorted.bam
bedtools bamtofastq -i trimmed_pass_sorted.bam -fq trimmed_pass_sorted.fastq
```

#### Visualizar calidad de las lecturas

```
fastqc trimmed_pass_sorted.fastq
```

