# MO_Pipeline

Una estandarización para la obtención de inputs para MaLiAmPi y la posterior ejecución del flujo de trabajo con datos públicos.

## 1. Descargar nextflow
(**Nota:** Nextflow requiere la instalación de java runtime enviroment, el desarrollador de MaLiAmPi, **Jonathan Golob**, sugiere de la versión [OpenJDK8](https://adoptopenjdk.net/) en adelante)
a) Descarga de Nextflow con el comando curl:
```
curl -s https://get.nextflow.io | bash
```
b) Prueba de Nextflow 
Para corroborar la correcta instalación de Nextflow corre el siguiente comando:
```
./nextflow run hello
```
Deberías obtener lo siguiente:
```
N E X T F L O W  ~  version 23.10.1
Launching `https://github.com/nextflow-io/hello` [agitated_koch] DSL2 - revision: 7588c46ffe [master]
executor >  local (4)
[59/ca68b0] process > sayHello (1) [100%] 4 of 4 ✔
Hola world!

Hello world!

Ciao world!

Bonjour world!
```
## 2. Descarga de Docker
Se sugiere el uso de este gestor de clusters para ejecutar este flujo de trabajo. Se sugiere seguir las instrucciones encontradas en el sitio de [Docker](https://docs.docker.com/engine/install/ubuntu/)

### Nota:
Se recomiendaactualizar las credenciales para el uso de Docker y posterior mente correr el comando
```
docker run hello-world
```
Si obtienes como output
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
significa la correcta instalación y el funcionamiento de las credenciales para Docker.

## 3. Configuración del archivo config para Nextflow
Para que Nextflow fluya suavemente debemos indicarle los recursos computacionales que tiene disponible. Para ello MaLiAmPi asigna mediante etiquetas el tipo de proceso y los recursos que ocupará:
  
  - 'io_limited': Procesos que no son multiprocesos, generalmente limitados por la velocidad de lectura y escritura de los archivos.
  - 'io_limited_local': Procesos limitados por Inputs/Outputs y deben ser ejecutados localmente.
  - 'io_mem': Procesos limitados por la velocidad de lectura y escritura de archivos pero requieren mucha memoria.
  - 'multithread': Procesos con requerimientos modestos de CPU y memoria. Pueden ser beneficiados por multiples CPUs y no son limitados por archivos de lectura y escritura.
  - 'mem_veryhigh': Tareas multiprocesos que requieren grandes cantidades de memoria.
  
De igual manera, en el archivo config se debe de especificar el uso de Docker o el sistema de manejo de clusters que se esté empleando.

El desarrollador aconseja el uso de la siguiente configuración del archivo config, el cual lo podemos copiar en la ruta `~/.nextflow/config`:
```groovy
profiles{
    standard {
        process {
            executor = 'local'
            withLabel: 'io_limited' {
                cpus = 1
                memory = 4.GB
            }
            withLabel: 'io_limited_local' {
                cpus = 1
                memory = 4.GB
            }
            withLabel: 'io_mem' {
                cpus = 1
                memory = 16.GB
            }
            withLabel: 'multithread' {
                cpus = 4
                memory = 16.GB
            }
            withLabel: 'mem_veryhigh' {
                cpus = 2
                memory = 32.GB
            }
        }
        docker {
            enabled = true
        }
    } // end standard profile
    hybrid {  // start hybrid profile combining local and awsbatch
        process {
            withLabel: 'io_limited' {
                cpus = 1
                memory = 4.GB
                executor = 'awsbatch'
            }
            withLabel: 'io_limited_local' {
                cpus = 1
                memory = 4.GB
                executor = 'local'
                docker.enabled = true
            }
            withLabel: 'io_mem' {
                cpus = 1
                memory = 16.GB
                executor = 'awsbatch'
            }
            withLabel: 'multithread' {
                cpus = 4
                memory = 16.GB
                executor = 'awsbatch'
            }
            withLabel: 'mem_veryhigh' {
                cpus = 2
                memory = 32.GB
                executor = 'awsbatch'
            }
        }
    } // end hybrid profile
}
```
Con esta configuración es posible manipular grupos de datos modestos, incluyendo los datos de prueba para el pipeline **MaLiAmPi**. Para datos de mayor tamaño se recomienda asignar más recursos computacionales al flujo de trabajo mediante el archivo **config**.

# Tutorial de MaLiAmPi
Con el setup descrito podemos hacer uso de los archivos de prueba de MaLiAmPi para familiarizarnos con el pipeline.

### 1. Obtención de datos de prueba
En el repositorio de [MaLiAmPi](https://github.com/jgolob/maliampi/tree/master) se encuentran los datos de prueba antes mencionados. Con los siguientes códigos obtendremos la versión actual del repositorio, guardaremos los archivos de prueba y borraremos lo demás.
```
wget https://github.com/jgolob/maliampi/archive/master.zip
unzip master.zip
mv maliampi-master/maliampi-practice-data/ .
rm -r maliampi-master && rm master.zip
cd maliampi-practice-data
```
### 2. Preparación de directorios
```
mkdir -p working/ && mkdir -p output/sv/
```
### 3. Analizar el archivo *manifest*
MaLiAmPi utiliza un archivo de manifiesto para "ubicar" los inputs que empleará. Este archivo solicita al menos dos columnas: ***specimen*** (contiene identificadores unicos para cada muestra) y ***R1*** (contiene las rutas absolutas de los archivos *fastq* de las secuencias *Forward* que se utilizarán). Además, podremos encontrar otra columna para las secuencias *Reverse*, ***R2***.

### 4. Usar nextflow para mostrar la línea de comandos de MaLiAmPi
```
nextflow run jgolob/maliampi -latest --help
```
Deberías de obtener el siguiente output:
```
N E X T F L O W  ~  version 23.10.1
Pulling jgolob/maliampi ...
 Already-up-to-date
Launching `https://github.com/jgolob/maliampi` [kickass_jang] DSL2 - revision: 333d83ba98 [master]
Usage:

nextflow run jgolob/maliampi <ARGUMENTS>

Required Arguments:
    --manifest            CSV file listing samples
                            At a minimum must have columns:
                                specimen: A unique identifier 
                                R1: forward read
                                R2: reverse read fq

                            optional columns:
                                batch: sequencing / library batch. Should be filename safe
                                I1: forward index file (for checking demultiplexing)
                                I2: reverse index file
    --repo_fasta          Repository of 16S rRNA genes.
    --repo_si             Information about the 16S rRNA genes.
    --email               Email (for NCBI)
Options:
  Common to all:
    --output              Directory to place outputs (default invocation dir)
                            Maliampi will create a directory structure under this directory
    -w                    Working directory. Defaults to `./work`
    -resume                 Attempt to restart from a prior run, only completely changed steps

SV-DADA2 options:
    --trimLeft              How far to trim on the left (default = 0)
    --maxN                  (default = 0)
    --maxEE                 (default = Inf)
    --truncLenF             (default = 0)
    --truncLenR             (default = 0)
    --truncQ                (default = 2)
    --minOverlap            (default = 12)
    --maxMismatch           (default = 0)


Ref Package options (defaults generally fine):
    --raxml                     Which raxml to use: og (original) or ng (new). Default: og
    --repo_min_id               Minimum percent ID to a SV to be recruited (default = 0.8)
    --repo_max_accepts          Maximum number of recruits per SV (default = 10)
    --cmalign_mxsize            Infernal cmalign mxsize (default = 8196)
    --raxml_model               RAxML model for tree formation (default = 'GTRGAMMA')
    --raxml_parsiomony_seed     (default = 12345)        
    --raxmlng_model             Subsitution model (default 'GTR+G')
    --raxmlng_parsimony_trees   How many seed parsimony trees (default 1)
    --raxmlng_random_trees      How many seed random trees (default 1)
    --raxmlng_bootstrap_cutoff  When to stop boostraps (default = 0.3)
    --raxmlng_seed              Random seed for RAxML-ng (default = 12345)
    --taxdmp                    (Optional) taxdmp.zip from the repository

Placement / Classification Options (defaults generally fine):
    --pp_classifer                  pplacer classifer (default = 'hybrid2')
    --pp_likelihood_cutoff          (default = 0.9)
    --pp_bayes_cutoff               (default = 1.0)
    --pp_multiclass_min             (default = 0.2)
    --pp_bootstrap_cutoff           (default = 0.8)
    --pp_bootstrap_extension_cutoff (default = 0.4)
    --pp_nbc_boot                   (default = 100)
    --pp_nbc_target_rank            (default = 'genus')
    --pp_nbc_word_length            (default = 8)
    --pp_seed                       (default = 1)
```
Para poder funcionar, MaLiAmPi requiere como mínimo los siguientes argumentos:
- --manifest: Archivo .csv que contiene las rutas absolutas de las secuencias en formato fastq.
- --repo_fasta: Archivo fasta con genes de ARNr 16S de referencia.
- --repo_si: Archivo .csv que vincula los IDs de las secuencias fasta de referencia con la taxonomía de NCBI.
- --email: Dirección de correo para usar en el NCBI.

#### 4.1 Modificación del archivo ***main.nf***
El desarrollador del pipeline ([Jonathan Golob](https://github.com/jgolob)) indica que existe un bug en la utilidad gappa dentro del módulo EPA-NG place, por lo tanto sugiere que utilicemos el modulo pplacer_place_classify.nf en su lugar. Para ello:
a) Haremos una copia del flujo de trabajo 
```
cp -r /home/usr/.nextflow/assets/jgolob/maliampi maliampi_pplacer
```
b) Modificar el archivo ***main.nf***
Con el comando
```
nano /home/usr/.nextflow/assets/jgolob/maliampi_pplacer
```
Modificaremos dentro del apartado ```//Modules ``` 
```
include { epang_place_classify_wf } from './modules/epang_place_classify' params 
```
por
```
include { pplacer_place_classify_wf} from './modules/pplacer_place_classify' params
```
A su vez, en el apartado ```STEP 3. Place and Classify``` modificaremos
```
epang_place_classify_wf(
        dada2_wf.out.sv_fasta,
        make_refpkg_wf.out.refpkg_tgz,
        dada2_wf.out.sv_long
    )

```
por
```
pplacer_place_classify_wf(
        dada2_wf.out.sv_fasta,
        make_refpkg_wf.out.refpkg_tgz,
        dada2_wf.out.sv_weights,
	dada2_wf.out.sv_map
    )

```
Con esta modificación podemos correr MaLiAmPi. Para corroborarlo se ejecuta el siguiente comando:
```
nextflow run /home/usr/.nextflow/assets/jgolob/maliampi_pplacer -latest --help
```
Si se modificó correctamente el archivo ***main.nf*** debería obtener el siguiente output:
```
N E X T F L O W  ~  version 23.10.1
Launching `/home/usr/.nextflow/assets/jgolob/maliampi_pplacer/main.nf` [voluminous_avogadro] DSL2 - revision: 987b9d9d14

Usage:

nextflow run jgolob/maliampi <ARGUMENTS>

Required Arguments:
    --manifest            CSV file listing samples
                            At a minimum must have columns:
                                specimen: A unique identifier 
                                R1: forward read
                                R2: reverse read fq

                            optional columns:
                                batch: sequencing / library batch. Should be filename safe
                                I1: forward index file (for checking demultiplexing)
                                I2: reverse index file
    --repo_fasta          Repository of 16S rRNA genes.
    --repo_si             Information about the 16S rRNA genes.
    --email               Email (for NCBI)
Options:
  Common to all:
    --output              Directory to place outputs (default invocation dir)
                            Maliampi will create a directory structure under this directory
    -w                    Working directory. Defaults to `./work`
    -resume                 Attempt to restart from a prior run, only completely changed steps

SV-DADA2 options:
    --trimLeft              How far to trim on the left (default = 0)
    --maxN                  (default = 0)
    --maxEE                 (default = Inf)
    --truncLenF             (default = 0)
    --truncLenR             (default = 0)
    --truncQ                (default = 2)
    --minOverlap            (default = 12)
    --maxMismatch           (default = 0)


Ref Package options (defaults generally fine):
    --raxml                     Which raxml to use: og (original) or ng (new). Default: og
    --repo_min_id               Minimum percent ID to a SV to be recruited (default = 0.8)
    --repo_max_accepts          Maximum number of recruits per SV (default = 10)
    --cmalign_mxsize            Infernal cmalign mxsize (default = 8196)
    --raxml_model               RAxML model for tree formation (default = 'GTRGAMMA')
    --raxml_parsiomony_seed     (default = 12345)        
    --raxmlng_model             Subsitution model (default 'GTR+G')
    --raxmlng_parsimony_trees   How many seed parsimony trees (default 1)
    --raxmlng_random_trees      How many seed random trees (default 1)
    --raxmlng_bootstrap_cutoff  When to stop boostraps (default = 0.3)
    --raxmlng_seed              Random seed for RAxML-ng (default = 12345)
    --taxdmp                    (Optional) taxdmp.zip from the repository

Placement / Classification Options (defaults generally fine):
    --pp_classifer                  pplacer classifer (default = 'hybrid2')
    --pp_likelihood_cutoff          (default = 0.9)
    --pp_bayes_cutoff               (default = 1.0)
    --pp_multiclass_min             (default = 0.2)
    --pp_bootstrap_cutoff           (default = 0.8)
    --pp_bootstrap_extension_cutoff (default = 0.4)
    --pp_nbc_boot                   (default = 100)
    --pp_nbc_target_rank            (default = 'genus')
    --pp_nbc_word_length            (default = 8)
    --pp_seed                       (default = 1)
```
Con el archivo modificado, podremos correr MaLiAmPi sin problemas.

### 5.Correr MaLiAmPi con los datos de prueba
```
nextflow run /home/usr/.nextflow/assets/jgolob/maliampi_pplacer -latest -resume --manifest manifest.csv --output tutorial_output --email your@email.com --repo_fasta test.repo.fasta --repo_si test.repo.seq_info.csv
```
Si todo funciona correctamente, deberás obtener el siguiente output en tu pantalla:
```
N E X T F L O W  ~  version 23.10.1
Launching `/home/usr/.nextflow/assets/jgolob/maliampi_pplacer/main.nf` [stoic_noether] DSL2 - revision: 987b9d9d14
[d8/8bca19] process > preprocess_wf:barcodecop (9)                    [100%] 9 of 9 ✔
[89/100945] process > preprocess_wf:FastQC_Raw (17)                   [100%] 18 of 18 ✔
[df/8e6309] process > preprocess_wf:TrimGalore (9)                    [100%] 9 of 9 ✔
[-        ] process > preprocess_wf:TrimGaloreSE                      -
[8f/cc2019] process > dada2_wf:dada2_ft (9)                           [100%] 9 of 9 ✔
[-        ] process > dada2_wf:dada2_ft_se                            -
[-        ] process > dada2_wf:dada2_ft_pyro                          -
[c1/91e207] process > dada2_wf:FastQC_PostFT (18)                     [100%] 18 of 18 ✔
[71/4c18ef] process > dada2_wf:dada2_derep (9)                        [100%] 9 of 9 ✔
[-        ] process > dada2_wf:dada2_derep_se                         -
[-        ] process > dada2_wf:dada2_derep_pyro                       -
[aa/a729ec] process > dada2_wf:dada2_learn_error (6)                  [100%] 6 of 6 ✔
[-        ] process > dada2_wf:dada2_learn_error_pyro                 -
[8b/ef4a8f] process > dada2_wf:dada2_derep_batches (6)                [100%] 6 of 6 ✔
[-        ] process > dada2_wf:dada2_derep_batches_pyro               -
[82/0adb68] process > dada2_wf:dada2_dada (6)                         [100%] 6 of 6 ✔
[-        ] process > dada2_wf:dada2_dada_pyro                        -
[32/cbe72e] process > dada2_wf:dada2_demultiplex_dada (6)             [100%] 6 of 6 ✔
[d7/8b474a] process > dada2_wf:dada2_merge (9)                        [100%] 9 of 9 ✔
[8f/afbceb] process > dada2_wf:dada2_seqtab_sp (9)                    [100%] 9 of 9 ✔
[a5/00b9d2] process > dada2_wf:dada2_seqtab_combine_batch (3)         [100%] 3 of 3 ✔
[4d/077a43] process > dada2_wf:dada2_seqtab_combine_all               [100%] 1 of 1 ✔
[15/b27f1a] process > dada2_wf:dada2_remove_bimera                    [100%] 1 of 1 ✔
[44/bcbd60] process > dada2_wf:Dada2_convert_output                   [100%] 1 of 1 ✔
[38/bf8132] process > output_failed                                   [100%] 1 of 1 ✔
[11/6bbdbe] process > make_refpkg_wf:RefpkgSearchRepo                 [100%] 1 of 1 ✔
[32/2ad4c7] process > make_refpkg_wf:FilterSeqInfo                    [100%] 1 of 1 ✔
[fd/ab1b2b] process > make_refpkg_wf:DlBuildTaxtasticDB               [100%] 1 of 1 ✔
[0a/9c117a] process > make_refpkg_wf:ConfirmSI                        [100%] 1 of 1 ✔
[a5/1dbfd3] process > make_refpkg_wf:RemoveDroppedRecruits            [100%] 1 of 1 ✔
[8b/0f0858] process > make_refpkg_wf:CombinedRefFilter                [100%] 1 of 1 ✔
[c3/2145d1] process > make_refpkg_wf:AlignRepoRecruits                [100%] 1 of 1 ✔
[b0/7a842d] process > make_refpkg_wf:ConvertAlnToFasta                [100%] 1 of 1 ✔
[e7/03e1e7] process > make_refpkg_wf:TaxtableForSI                    [100%] 1 of 1 ✔
[d8/c9dc3e] process > make_refpkg_wf:RaxmlTree                        [100%] 1 of 1 ✔
[8d/ccf13d] process > make_refpkg_wf:RaxmlTree_cleanupInfo            [100%] 1 of 1 ✔
[0e/407cda] process > make_refpkg_wf:CombineRefpkg_og                 [100%] 1 of 1 ✔
[f6/41f200] process > pplacer_place_classify_wf:ExtractRefpkg         [100%] 1 of 1 ✔
[ce/22e239] process > pplacer_place_classify_wf:AlignSV               [100%] 1 of 1 ✔
[da/a6c9a8] process > pplacer_place_classify_wf:CombineAln_SV_refpkg  [100%] 1 of 1 ✔
[7e/6de047] process > pplacer_place_classify_wf:PplacerPlacement      [100%] 1 of 1 ✔
[d7/d28f94] process > pplacer_place_classify_wf:PplacerReduplicate    [100%] 1 of 1 ✔
[f7/204952] process > pplacer_place_classify_wf:PplacerADCL           [100%] 1 of 1 ✔
[0e/82676a] process > pplacer_place_classify_wf:PplacerEDPL           [100%] 1 of 1 ✔
[92/b2d70e] process > pplacer_place_classify_wf:PplacerPCA            [100%] 1 of 1 ✔
[05/59f54d] process > pplacer_place_classify_wf:PplacerAlphaDiversity [100%] 1 of 1 ✔
[7c/f6647c] process > pplacer_place_classify_wf:PplacerKR             [100%] 1 of 1 ✔
[04/be45ce] process > pplacer_place_classify_wf:ClassifyDB_Prep       [100%] 1 of 1 ✔
[83/418160] process > pplacer_place_classify_wf:ClassifySV            [100%] 1 of 1 ✔
[01/878c6a] process > pplacer_place_classify_wf:ClassifyMCC           [100%] 1 of 1 ✔
[fb/73b00b] process > pplacer_place_classify_wf:ClassifyTables (4)    [100%] 6 of 6 ✔
[79/391236] process > pplacer_place_classify_wf:Extract_Taxonomy      [100%] 1 of 1 ✔
Completed at: 01-Mar-2024 19:14:37
Duration    : 3m 42s
CPU hours   : 1.4
Succeeded   : 153
```

### 6. Revisar el output
En el directorio asignado como output deberás encontrar los siguientes subdirectorios:
```
tutorial_output/
├── classify
├── placement
├── refpkg
└── sv
```
Para información respecto a su contenido se recomienda visitar el repositorio de [MaLiAmPi](https://github.com/jgolob/maliampi/tree/master).

# Uso de MaLiAmPi con datos reales
Una de las razones de la creación de MaLiAmPi hacer comparables los datos de diferentes estudios de microbioma y para ello se emplean las secuencias **fastq** de dichos estudios. 
En la esta sección se abordará la **descarga de secuencias fastq** de repositorios públicos así como la **creación del manifiesto** y la **obtención de secuencias de referencias**.

### 1. Obtención de secuencias de repositorios públicos
Los estudios que generan datos de secuenciación públicos los suben al ***Sequence Read Archive (SRA)***, una base de datos pública que forma parte del NCBI. A su vez, nos proporcionan el kit de herramientas  **SRA Toolkit** para facilitar la descarga de las secuencias fastq.

#### a)Descarga y descompresión de la paquetería SRA Toolkit
```
wget https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/3.0.10/sratoolkit.3.0.10-ubuntu64.tar.gz

tar -xzvf sratoolkit.3.0.10-ubuntu64.tar.gz
```
Deberías obtener los siguientes directorios:
```
sratoolkit.3.0.10-ubuntu64/
├── bin
├── CHANGES
├── example
├── README-blastn
├── README-vdb-config
├── README.md
└── schema
```
Dentro del directorio ```bin``` se encontrarán los ejecutables que se emplearán.
**NOTA IMPORTANTE:** Para descargar las secuencias fastq.gz necesitarás una ***Accesion_List*** generada en [SRA Run Selector](https://www.ncbi.nlm.nih.gov/Traces/study/), una vez la tengas podrás pasar a lo siguiente.

#### b) Descarga de secuencias en formato SRA
Las secuencias deben descargarse primero en formato SRA mediante el comando ```prefetch```. Se recomienda la generación de un archivo ejecutable como el siguiente:
```
#!/bin/bash

# Leer el archivo de texto línea por línea
while IFS= read -r id; do
    # Ejecutar prefetch para cada ID
    /home/usr/sratoolkit.3.0.10-ubuntu64/bin/prefetch $id
done < /home/usr/reads/SRR_Acc_List.txt

```
Esto generará un directorio para cada muestra que contendrá el archivo .sra:
```
SRRXXXXXXX/
└── SRRXXXXXXX.sra

```

#### c) Obtención de secuencias en formato fastq.gz
Para transformar los archivos SRA en fastq se empleará el comando ```fasterq-dump```. A su vez, se usará el comando ```gzip``` para comprimir las secuencias a fotmato ***fastq.gz*** debido a que así lo requiere MaLiAmPi. 
```
#!/bin/bash

# Leer el archivo de texto línea por línea
while IFS= read -r id; do
    # Ejecutar fastq-dump para cada ID
    /home/usr/sratoolkit.3.0.10-ubuntu64/bin/fasterq-dump -3 --skip-technical $id
        # Comprimir archivos fastq generados
    gzip ${id}_1.fastq
    gzip ${id}_2.fastq
    # Si hay archivos huérfanos, también comprimirlos
    if [ -e ${id}.fastq ]; then
        gzip ${id}.fastq
    fi
done < /home/usr/reads/SRR_Acc_List.txt
```
**NOTA:** Los archivos ejecutables se emplean de la siguiente manera:
```
bash archivo_ejecutable.sh
```
Esto generará 3 archivos fastq:
```
sratoolkit.3.0.10-ubuntu64/
├── SRRXXXXXXX.fastq   #Lecturas filtradas
├── SRRXXXXXXX_1.fastq #Lecturas Forward
└── SRRXXXXXXX_2.fastq #Lecturas Reverse
```

### 2. Generación de manifiesto
#### a)Generación de rutas absolutas Forward (R1) y Reverse (R2)
Para obtener las rutas absolutas corra el siguiente comando en el directorio donde se encuentran las secuencias fastq:
```
ls -d $PWD/*_1.* > ./reads_R1.txt
ls -d $PWD/*_2.* > ./reads_R2.txt
```

#### b) Generación del manifiesto
```
awk -F '[_/-]' '{print $13}' reads_R1.txt | paste -d ',' /dev/stdin reads_R1.txt | paste -d ',' /dev/stdin reads_R2.txt | awk 'BEGIN {OFS=","; print "specimen", "R1","R2" } {print $0}' > Manifiesto.csv
```
Deberiamos tener un archivo .csv con la siguiente estructura:
```
| specimen     | R1                                 |   R2                              |
|--------------|------------------------------------|-----------------------------------|
| SRRXXXXXX1   | /home/usr/reads/SRRXXXXXX1_1.fastq | /home/usr/reads/SRRXXXXXX1_2.fastq|
| SRRXXXXXX2   | /home/usr/reads/SRRXXXXX2_1.fastq  | /home/usr/reads/SRRXXXXXX2_2.fastq|
| SRRXXXXXX3   | /home/usr/reads/SRRXXXXXX3_1.fastq | /home/usr/reads/SRRXXXXXX3_2.fastq|
```

### 3. Obtención de secuencias de referencia

En el link https://zenodo.org/records/6876634/files/arf_20200420.tgz se encuentra un archivo .tgz . 

  a) Obtención y descompresión del archivo .tgz:
```bash
wget https://zenodo.org/records/6876634/files/arf_20200420.tgz

tar -xvzf arf_20200420.tgz 
```
Con ello obtendremos el directorio ```arf_20200420``` que contendrá los siguientes archivos:
```
arf_20200420/
├── dedup
│   └── 1200bp
│       ├── named
│       │── ├── blast.nhr
│       │── ├── blast.nin
│       │── ├── blast.nsq
│       │── ├── filtered
│       │── │   ├── blast.nhr
│       │── │── ├── blast.nin
│       │── │   ├── blast.nsq
│       │── │   ├── lineages.csv
│       │── │   ├── lineages.txt
│       │── │   ├── outliers.csv
│       │── │   ├── seq_info.csv
│       │── │   ├── seqs.fasta
│       │── │   └── taxonomy.csv
│       │── ├── lineages.csv
│       │── ├── lineages.txt
│       │── ├── seq_info.csv
│       │── ├── seqs.fasta
│       │── └── taxonomy.csv
│       ├── seq_info.csv
│       ├── seqs.fasta
│       └── types
│           ├── blast.nhr
│           ├── blast.nin
│           ├── blast.nsq
│           ├── lineages.csv
│           ├── lineages.txt
│           ├── seq_info.csv
│           ├── seqs.fasta
│           └── taxonomy.csv
├── pubmed_info.csv
├── records.txt
├── references.csv
├── refseq_info.csv
├── seq_info.csv
├── seqs.fasta
├── taxdmp.zip
├── taxonomy.csv
└── taxonomy.db
```

## Nota:

La carpeta ***arf_20200420*** contiene las secuencias extraidas del NCBI mediante el pipeline ***ya16sdb***. ARF se encarga de filtrar las secuencias de dicha base de datos (o la base de datos que usemos como input) a distintos niveles, donde ***1200*** hace referencia a secuencias con más de 1200 pb de longitud, ***named*** contiene asignaciones a nivel de especie y, finalmente, ***filtered*** contiene solo secuencias que no son outliers.

Los documentos ***seq_info.csv*** y ***seqs.fasta*** son inputs necesarios para MaLiAmPi. En este sentido **se sugiere utilizar aquellos que se encuentran en la carpeta** ```filtered```.

