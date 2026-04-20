# Runbook — comandos completos

✅ **Isto não é um script.**  
Copie e execute os comandos **um a um**, validando impacto e contexto.


<details>
<summary><strong>➡️ Habilitar execução de scripts customizados do VMware Tools</strong></summary>

```bash
################################################################################
# Habilitar execução de scripts customizados do VMware Tools
################################################################################

# Verifica se scripts customizados estão habilitados no VMware Tools
vmware-toolbox-cmd config get deployPkg enable-custom-scripts

# Habilita a execução de scripts customizados
vmware-toolbox-cmd config set deployPkg enable-custom-scripts true
```

</details>


<details>
<summary><strong>➡️ Identificação de dispositivos de storage (Linux)</strong></summary>

```bash
################################################################################
# Identificação de dispositivos de storage (Linux)
################################################################################

# Lista dispositivos de bloco com informações detalhadas
lsblk -o NAME,SIZE,TYPE,FSTYPE,MODEL,SERIAL,WWN,VENDOR

# Exibe informações completas do device via udev
udevadm info --query=all --name=/dev/sdb
```

</details>


<details>
<summary><strong>➡️ LVM — Remoção e recriação de Logical Volumes (RHEL 9)</strong></summary>

```bash
################################################################################
# LVM — Remoção e recriação de Logical Volumes (RHEL 9)
################################################################################

# Exibe detalhes do Logical Volume lun03
lvdisplay /dev/vg_stg/lun03

# Desativa o LV para permitir remoção segura
lvchange -an /dev/vg_stg/lun03

# Confirma que o LV está inativo
lvdisplay /dev/vg_stg/lun03

# Remove o Logical Volume (AÇÃO IRREVERSÍVEL)
lvremove -y /dev/vg_stg/lun03

# Valida que o LV foi removido
lvdisplay /dev/vg_stg/lun03

# Exibe informações de espaço do Volume Group
vgs vg_stg

# Lista Logical Volumes existentes
lvs vg_stg

# Cria Logical Volume lun03 com 5TB
lvcreate -L 5T -n lun03 vg_stg

# Cria Logical Volume lun04 com 5TB
lvcreate -L 5T -n lun04 vg_stg

# Lista LVs com atributos completos
lvs -o vg_name,lv_name,lv_size,lv_path,lv_attr vg_stg

# Confirma devices no device-mapper
ls -l /dev/vg_stg/
```

</details>


<details>
<summary><strong>➡️ LVM — Expansão do filesystem /home adicionando novo disco (VMware)</strong></summary>

```bash
################################################################################
# CONTEXTO: /home cheio, disco novo adicionado no VMware para expansão via LVM
################################################################################

# Cria tabela de partição GPT no novo disco
parted -s /dev/sdg mklabel gpt

# Cria partição ocupando 100% do disco
parted -s /dev/sdg mkpart primary 0% 100%

# Marca a partição como destinada ao LVM
parted -s /dev/sdg set 1 lvm on

# Confirma o novo disco e partições
lsblk

# Lista Physical Volumes existentes
pvs

# Inicializa a partição como Physical Volume
pvcreate /dev/sdg1

# Confirma criação do PV
pvs

# Lista Volume Groups
vgs

# Adiciona o novo PV ao VG do /home
vgextend vg_ftp /dev/sdg1

# Confirma PVs e VGs após extensão
pvs
vgs

# Lista Logical Volumes
lvs

# Expande o Logical Volume usando todo o espaço livre
lvextend -l +100%FREE /dev/mapper/vg_ftp-vol_ftp

# Confirma novo tamanho do LV
lvs

# Mostra uso do filesystem antes da expansão
df -h

# Expande filesystem XFS de forma online
xfs_growfs /home

# Valida novo tamanho do filesystem
df -h
```

</details>
