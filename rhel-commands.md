################################################################################
# Habilitar execução de scripts customizados do VMware Tools
#
# Permite que a VM execute scripts de customização durante o deploy ou clonagem
# (Guest OS Customization)
################################################################################

# Verifica configuração atual
vmware-toolbox-cmd config get deployPkg enable-custom-scripts

# Habilita execução de scripts customizados
vmware-toolbox-cmd config set deployPkg enable-custom-scripts true


################################################################################
# Identificação de dispositivos de storage (Linux)
################################################################################

# Lista detalhes do dispositivo para identificar origem
# (vendor, modelo, serial, WWN)
lsblk -o NAME,SIZE,TYPE,FSTYPE,MODEL,SERIAL,WWN,VENDOR

# Mostra informações completas do device via udev
# (bus, path, FC, WWN)
udevadm info --query=all --name=/dev/sdb


################################################################################
# LVM — Remoção e recriação de Logical Volumes (RHEL 9)
################################################################################

# Exibe todas as informações do Logical Volume lun03
# Usado para confirmar tamanho, status e detalhes antes de qualquer alteração
lvdisplay /dev/vg_stg/lun03

# Desativa o Logical Volume lun03
# Garante que o volume não esteja em uso antes da remoção
lvchange -an /dev/vg_stg/lun03

# Confirma que o LV lun03 está desativado (NOT available)
lvdisplay /dev/vg_stg/lun03

# Remove definitivamente o Logical Volume lun03
# ATENÇÃO: esta ação é irreversível
lvremove -y /dev/vg_stg/lun03

# Valida que o Logical Volume lun03 não existe mais
lvdisplay /dev/vg_stg/lun03

# Mostra o Volume Group vg_stg e o espaço livre após a remoção do LV
vgs vg_stg

# Lista os Logical Volumes existentes no Volume Group vg_stg
lvs vg_stg

# Cria o Logical Volume lun03 com 5 TB
lvcreate -L 5T -n lun03 vg_stg

# Cria o Logical Volume lun04 com 5 TB
lvcreate -L 5T -n lun04 vg_stg

# Lista os LVs com detalhes
# (VG, nome, tamanho, path e atributos)
lvs -o vg_name,lv_name,lv_size,lv_path,lv_attr vg_stg

# Mostra os device links criados em /dev/vg_stg/
# Confirma o mapeamento para os device-mapper (dm-*)
ls -l /dev/vg_stg/