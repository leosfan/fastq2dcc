# FASTQ2DCC
Install the GeoMxNGSPipeline_Linux_2.0.0.16 tool from Nanostring/illumina to docker container, and convert it to Singularity to better adapt a non-root environment in HPC.

## Tested on WSL2 CentOS 7
```bash
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

## Installation

### GeoMxNGSPipeline_Linux_2.0.0.16.zip

#### pull the base docker container
```bash
docker pull ubuntu:18.04
```
#### install GeoMxNGSPipeline_Linux
```bash
docker run -dit --name ubuntu ubuntu:18.04
docker start `docker ps -q -l`
docker cp ../tools/GeoMxNGSPipeline_Linux_2.0.0.16.zip ubuntu:/var
docker exec -it --privileged ubuntu bash
apt-get update
apt-get install unzip
unzip GeoMxNGSPipeline_Linux_2.0.0.16.zip
bash /var/GeoMxNGSPipeline_Linux_2.0.0.16.sh
chmod +x /var/GeoMxNGSPipeline/geomxngspipeline
echo 'export DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1' >> ~/.bashrc
source ~/.bashrc
```

#### save docker image
```bash
docker commit ubuntu dsp_rna_fastq2dcc:latest
```

### convert docker image to singularity
#### push to local registry
```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
docker run -dit -p 6000:5000 --name dsp_rna_fastq2dcc  dsp_rna_fastq2dcc:latest
docker tag dsp_rna_fastq2dcc:latest localhost:5000/dsp_rna_fastq2dcc:latest
docker push localhost:5000/dsp_rna_fastq2dcc:latest
```

#### create singularity imagedock
```bash
# using the def file in repo
sudo SINGULARITY_NOHTTPS=true singularity build dsp_rna_fastq2dcc.simg
```

### test if the image is successfully installed
```bash
singularity exec ./dsp_rna_fastq2dcc.simg /var/GeoMxNGSPipeline/geomxngspipeline
```

#### run conversion
```bash
# the tool will use 20 cores regardless of the  threads option
nohup singularity exec --bind /mnt:/path/to/workdir /home/fanshiheng/container/dsp_rna_fastq2dcc.simg /var/GeoMxNGSPipeline/geomxngspipeline --in /path/to/workdir/fastq --out /path/to/workdirdcc --ini /path/to/workdir/YKKY0012-20220727_20220802T0545_GNP_config.ini --threads=40 &
```

#### check the resource sonsumption of singularity when executing
```bash
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | grep "singularity exec "
```
