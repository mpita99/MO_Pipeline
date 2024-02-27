# MO_Pipeline

Una estandarización para la obtención de inputs para MaLiAmPi y la posterior ejecución del flujo de trabajo con datos públicos.

**1. Obtención del archivo arf**

En el link https://zenodo.org/records/6876634/files/arf_20200420.tgz se encuentra un archivo .tgz 
necesario para el funcionamiento de ARF. 

  a). Para obtener el archivo. tgz:
```bash
wget https://zenodo.org/records/6876634/files/arf_20200420.tgz
```
  b). Extraer los archivos del documento 
```bash
tar -xvzf arf_20200420.tgz 
```
**2.Extracción de secuencias mediante ARF**
```bash
NXF_VER=22.10.0 nextflow run jgolob/arf --repo /arf_20200420 --out arf_repo_reads --email your@email.com --min_len 1200 --ncbi_concurrent_connections 8 --retry_max 3 --api_key your.api.key
```

