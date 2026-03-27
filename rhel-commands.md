## Habilitar execução de scripts customizados do VMware Tools

Permite que a VM execute scripts de customização durante o deploy ou clonagem  
(**Guest OS Customization**).

---

## Runbook — comandos completos

> ✅ Use este bloco quando precisar copiar e colar tudo de uma vez

<details>
<summary><strong>➡️ Clique aqui para expandir e copiar todos os comandos</strong></summary>

```bash
################################################################################
# Habilitar execução de scripts customizados do VMware Tools
################################################################################

# Verifica configuração atual
vmware-toolbox-cmd config get deployPkg enable-custom-scripts

# Habilita execução de scripts customizados
vmware-toolbox-cmd config set deployPkg enable-custom-scripts true


################################################################################
# Identificação de dispositivos de storage (Linux)
################################################################################

# Lista detalhes do dispositivo (vendor, modelo, serial, WWN)
lsblk -o NAME,SIZE,TYPE,FSTYPE,MODEL,SERIAL,WWN,VENDOR

# Mostra informações completas do device via udev
udevadm info --query=all --name=/dev/sdb


################################################################################
# LVM — Remoção e recriação de Logical Volumes (RHEL 9)
################################################################################

# Inspeciona o Logical Volume lun03
lvdisplay /dev/vg_stg/lun03

# Desativa o Logical Volume
lvchange -an /dev/vg_stg/lun03

# Confirma status
lvdisplay /dev/vg_stg/lun03

# Remove o Logical Volume (AÇÃO IRREVERSÍVEL)
lvremove -y /dev/vg_stg/lun03

# Valida remoção
lvdisplay /dev/vg_stg/lun03

# Verifica espaço livre no VG
vgs vg_stg

# Lista LVs existentes
lvs vg_stg

# Cria novos Logical Volumes
lvcreate -L 5T -n lun03 vg_stg
lvcreate -L 5T -n lun04 vg_stg

# Lista LVs com detalhes completos
lvs -o vg_name,lv_name,lv_size,lv_path,lv_attr vg_stg

# Confirma device-mapper
ls -l /dev/vg_stg/