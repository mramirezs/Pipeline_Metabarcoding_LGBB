# Pipeline Metabarcoding

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

## Ejecutar POD5

```
pod5 convert fast5 fast5/*.fast5 --output converted.pod5

```

## Descargar modelos con dorado

```
dorado download
```

## Realizamos el llamado de las bases

```
dorado basecaller sup converted.pod5 > calls.bam

## Realizamos el trimado de las secuencias

```
dorado trim calls.bam > trimmed.bam
```

## Obtuvimos el resumen de la secuenciación 

```
dorado summary trimmed.bam > summary.tsv 

```
## Convertimos de bam a fastq

```
samtools sort -n trimmed.bam -o trimmed_sorted.bam
bedtools bamtofastq -i trimmed_sorted.bam -fq trimmed_sorted.fastq
```

## Remoción de Adaptadores y Control de Calidad

Usemos Porechop (v0.2.4) para la remoción de adaptadores.

```bash
ml porechop
porechop-runner.py -i basecalled/ -o trimmed/
```


