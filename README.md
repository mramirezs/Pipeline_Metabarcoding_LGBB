# Pipeline Metabarcoding

## Carpeta de trabajo de proyecto

`/media/prosopis/E/LGBB/Server/user/fast5`

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
mkdir DB
cd DB
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
seqkit stats -a -j 4 *_trimmed.fastq > stats_trimmed_fastq.txt
```
> **Comentario:** 
> - `-i`: Especifica el archivo de entrada.
> - `-o`: Especifica el archivo de salida después de la remoción de adaptadores.

```
file                                format  type  num_seqs      sum_len  min_len  avg_len  max_len       Q1     Q2       Q3  sum_gap    N50  Q20(%)  Q30(%)
SQK-16S024_barcode01_trimmed.fastq  FASTQ   DNA      2,232    2,806,626        2  1,257.4    3,808    1,225  1,404  1,453.5        0  1,419   50.81    8.49
SQK-16S024_barcode02_trimmed.fastq  FASTQ   DNA      1,622    1,899,710        4  1,171.2    1,949      984  1,387    1,451        0  1,412   50.58    8.59
SQK-16S024_barcode03_trimmed.fastq  FASTQ   DNA     12,734   15,286,054        3  1,200.4    2,838    1,089  1,391    1,441        0  1,405   50.68    7.81
SQK-16S024_barcode04_trimmed.fastq  FASTQ   DNA      8,438    9,920,380        8  1,175.7    2,091    1,053  1,386    1,432        0  1,399   50.35     7.5
SQK-16S024_barcode05_trimmed.fastq  FASTQ   DNA        678      868,972       59  1,281.7    1,937    1,333  1,402    1,454        0  1,415   50.02    7.34
SQK-16S024_barcode06_trimmed.fastq  FASTQ   DNA      3,132    3,986,918        3    1,273    2,216    1,322  1,406    1,448        0  1,416   51.28    7.43
SQK-16S024_barcode07_trimmed.fastq  FASTQ   DNA     12,376   14,725,908        5  1,189.9    2,838    1,060  1,392    1,442        0  1,406   50.58    7.45
SQK-16S024_barcode08_trimmed.fastq  FASTQ   DNA        312      436,068       28  1,397.7    1,835  1,388.5  1,419    1,453        0  1,422    50.5    8.42
SQK-16S024_barcode09_trimmed.fastq  FASTQ   DNA     22,568   18,558,358        2    822.3    1,961      366    886    1,336        0  1,255   51.23    7.88
SQK-16S024_barcode10_trimmed.fastq  FASTQ   DNA     17,682   19,702,542        3  1,114.3    2,647      723  1,311    1,404        0  1,387   50.87     8.2
SQK-16S024_barcode11_trimmed.fastq  FASTQ   DNA     13,658   14,513,508        1  1,062.6    2,029      838  1,192    1,383        0  1,336    50.9    7.86
SQK-16S024_barcode12_trimmed.fastq  FASTQ   DNA     11,828   13,383,462       10  1,131.5    1,996      950  1,334    1,398        0  1,374   50.41    7.71
SQK-16S024_barcode13_trimmed.fastq  FASTQ   DNA     13,488   15,178,236        1  1,125.3    2,675    915.5  1,323    1,396        0  1,370   50.68    7.65
SQK-16S024_barcode14_trimmed.fastq  FASTQ   DNA     16,168   15,937,992        2    985.8    2,320      518  1,129    1,391        0  1,356   50.64     8.2
SQK-16S024_barcode15_trimmed.fastq  FASTQ   DNA     14,306   15,960,920        1  1,115.7    2,434      842  1,338    1,398        0  1,384   50.63    7.35
SQK-16S024_barcode16_trimmed.fastq  FASTQ   DNA     14,086   15,558,454        2  1,104.5    2,054      819  1,305    1,400        0  1,379   50.69     7.9
SQK-16S024_barcode17_trimmed.fastq  FASTQ   DNA     17,846   18,802,674        2  1,053.6    2,988      642  1,313    1,398        0  1,378   50.28    7.08
SQK-16S024_barcode18_trimmed.fastq  FASTQ   DNA     15,846   16,559,930        2  1,045.1    2,088      560  1,305    1,398        0  1,385   50.53    7.87
unclassified_trimmed.fastq          FASTQ   DNA    467,038  643,878,250        1  1,378.6    3,966    1,386  1,398    1,416        0  1,400   53.51    8.43
```

Emplearemos Chopper (v0.8.0) en lugar de Nanofilt para filtrar las secuencias por calidad y longitud

> **Comentario:** Chopper es una herramienta para filtrar lecturas de secuenciación de nanoporos por calidad y longitud.

```bash
ml chopper
for file in *_trimmed.fastq; do prefix="${file%_trimmed.fastq}"; chopper -i "${prefix}_trimmed.fastq" -c ppa_v2.asm.fasta  --quality 10 --minlength 1000 > "${prefix}_choppered.fastq"; done
seqkit stats -a -j 4 *_choppered.fastq
```

```
file                                  format  type  num_seqs      sum_len  min_len  avg_len  max_len       Q1       Q2     Q3  sum_gap    N50  Q20(%)  Q30(%)
SQK-16S024_barcode01_choppered.fastq  FASTQ   DNA      1,792    2,511,222    1,001  1,401.4    3,808    1,372  1,424.5  1,459        0  1,430   50.57    8.44
SQK-16S024_barcode02_choppered.fastq  FASTQ   DNA      1,172    1,627,376    1,002  1,388.5    1,949    1,354    1,419  1,460        0  1,426   50.45    8.44
SQK-16S024_barcode03_choppered.fastq  FASTQ   DNA      9,586   13,303,098    1,000  1,387.8    2,838    1,368    1,410  1,452        0  1,415   50.49    7.73
SQK-16S024_barcode04_choppered.fastq  FASTQ   DNA      6,190    8,555,270    1,000  1,382.1    2,091    1,359    1,404  1,448        0  1,407   49.78    7.31
SQK-16S024_barcode05_choppered.fastq  FASTQ   DNA        566      796,126    1,009  1,406.6    1,937    1,379    1,418  1,458        0  1,420   49.84    7.19
SQK-16S024_barcode06_choppered.fastq  FASTQ   DNA      2,578    3,628,286    1,001  1,407.4    2,216    1,387    1,420  1,453        0  1,424   51.17     7.4
SQK-16S024_barcode07_choppered.fastq  FASTQ   DNA      9,342   12,967,582    1,000  1,388.1    2,838    1,377    1,410  1,452        0  1,414   50.47    7.41
SQK-16S024_barcode08_choppered.fastq  FASTQ   DNA        300      422,882    1,031  1,409.6    1,835    1,392    1,421  1,454        0  1,424   50.02    8.21
SQK-16S024_barcode09_choppered.fastq  FASTQ   DNA      7,962   10,459,818    1,000  1,313.7    1,961    1,249    1,362  1,396        0  1,371   48.77    6.99
SQK-16S024_barcode10_choppered.fastq  FASTQ   DNA      9,870   14,414,240    1,000  1,460.4    2,647    1,339    1,396  1,582        0  1,403   49.76    8.02
SQK-16S024_barcode11_choppered.fastq  FASTQ   DNA      7,158    9,545,696    1,000  1,333.6    2,029    1,256    1,356  1,401        0  1,371    48.3    7.06
SQK-16S024_barcode12_choppered.fastq  FASTQ   DNA      7,104    9,779,448    1,000  1,376.6    1,996  1,329.5    1,385  1,408        0  1,389   48.57     7.3
SQK-16S024_barcode13_choppered.fastq  FASTQ   DNA      7,844   10,804,476    1,000  1,377.4    2,675    1,325    1,384  1,406        0  1,388   48.51    7.05
SQK-16S024_barcode14_choppered.fastq  FASTQ   DNA      7,712   10,628,336    1,000  1,378.2    2,320    1,323    1,387  1,408        0  1,391   48.74    7.72
SQK-16S024_barcode15_choppered.fastq  FASTQ   DNA      8,266   11,635,032    1,000  1,407.6    2,434    1,340    1,390  1,410        0  1,393   48.77    6.93
SQK-16S024_barcode16_choppered.fastq  FASTQ   DNA      7,998   11,255,078    1,000  1,407.2    2,054    1,328    1,389  1,413        0  1,395   48.96    7.53
SQK-16S024_barcode17_choppered.fastq  FASTQ   DNA      9,540   13,195,148    1,000  1,383.1    2,988    1,334    1,389  1,409        0  1,393   48.23    6.72
SQK-16S024_barcode18_choppered.fastq  FASTQ   DNA      8,302   11,785,060    1,000  1,419.5    2,088    1,341    1,392  1,413        0  1,395   48.78    7.44
unclassified_choppered.fastq          FASTQ   DNA    361,002  521,939,346    1,002  1,445.8    3,966    1,388    1,401  1,439        0  1,402   51.48    7.81
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
for file in *_choppered.fastq; do prefix="${file%_choppered.fastq}"; NanoPlot --fastq "${prefix}_choppered.fastq" --loglength -o "${prefix}_nanoplot"; done
```

> **Comentario:** 
> - `--fastq`: Especifica el archivo FASTQ de entrada.
> - `--loglength`: Genera gráficos con el logaritmo de la longitud de las lecturas.

## Clasificación taxonómica con Kraken

- Instalación:

```bash
cd /opt/apps
wget https://github.com/DerrickWood/kraken2/archive/v2.1.3.tar.gz  # Reemplaza v2.1.3 con la versión deseada
tar -xzvf v2.1.3.tar.gz

sudo apt-get update
sudo apt-get install build-essential libcurl4-openssl-dev zlib1g-dev

cd /opt/apps/kraken2
./install_kraken2.sh ../kraken2-2.1.3/

mkdir -p /opt/apps/modulefiles/Core/kraken2
nano 2.1.3.lua

-- -*- lua -*-
------------------------------------------------------------
-- kraken2 2.1.3
------------------------------------------------------------

help([[

        This module configures the environment for kraken2 2.1.3

]])


-- local variables
local version = "2.1.3"
local base = "/opt/apps/kraken2-" .. version .. "/"

whatis("Name: kraken2")
whatis("Version: 2.1.3")
whatis("Keywords: kraken2, Bioinformatics, clasification,taxonomy")
whatis("Category: Bioinformatics")
whatis("URL: https://ccb.jhu.edu/software/kraken/")
whatis("Description: Krake2n is a system for assigning taxonomic labels to short DNA sequences, usually obtained through metagenomic studies.")

-- Enviroment variables
setenv("NEXTFLOW_VE", version)
prepend_path("PATH", base)
```

Ingresamos como usuario y ejecutamos:

```
ml kraken2
kraken2 --help
```

Creamos la base de datos a partir del genoma de Neltuma pallida

```
mkdir neltuma_pallida_kraken2_db
mv ppa_v2.asm.fasta neltuma_pallida_kraken2_db/
sed 's/>/>204979 | /' neltuma_pallida_kraken2_db/ppa_v2.asm.fasta > neltuma_pallida_kraken2_db/ppa_v2_modificado.fasta
kraken2-build --db neltuma_pallida_kraken2_db --add-to-library neltuma_pallida_kraken2_db/ppa_v2_modificado.fasta
kraken2-build --download-taxonomy --db neltuma_pallida_kraken2_db/
kraken2-build --db neltuma_pallida_kraken2_db --build
```

Ejecución de kraken2:

```
for file in *_choppered.fastq; do prefix="${file%_choppered.fastq}"; kraken2 --db neltuma_pallida_kraken2_db --threads 8 --input "${prefix}_choppered.fastq" --output "${prefix}.kraken" --report "${prefix}.report"; done

wget https://raw.githubusercontent.com/marbl/Krona/master/scripts/kreport2krona.py

for file in *_choppered.fastq; do prefix="${file%_choppered.fastq}"; python kreport2krona.py -r "${prefix}.report" -o "${prefix}.krona"; done


wget https://github.com/marbl/Krona/archive/refs/tags/v2.8.1.tar.gz
tar xvfz v2.8.1.tar.gz
sudo ln -s /opt/apps/krona-2.8.1/KronaTools/scripts/ImportTaxonomy.pl /usr/local/bin/ktImportTaxonomy

cd KronaTools
./updateTaxonomy.sh

for file in *_choppered.fastq; do prefix="${file%_choppered.fastq}"; ktImportTaxonomy "${prefix}.krona" -o "${prefix}.html"; done
```

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
$ for file in *_choppered.fastq; do prefix="${file%_choppered.fastq}"; nextflow run /home/manuel.ramirez/Bioprograms/NanoCLUST/main.nf -profile docker --reads "${prefix}_choppered.fastq" --db /home/manuel.ramirez/Bioprograms/NanoCLUST/db/16S_ribosomal_RNA --tax /home/manuel.ramirez/Bioprograms/NanoCLUST/db/taxdb/ --outdir "${prefix}_nanoclust" --multiqc; done
```

> **Comentario:** 
> - `--reads`: Especifica el archivo de lecturas FASTQ de entrada.
> - `--db`: Especifica la base de datos de 16S ribosomal RNA.
> - `--tax`: Especifica la base de datos de taxonomía.
> - `--outdir`: Especifica el directorio de salida para los resultados.
> - `--multiqc`: Genera un informe de calidad múltiple con MultiQC.
