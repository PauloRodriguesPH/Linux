
# **Transformando um Dell NX3200 em um Storage Fibre Channel usando SCST + QLogic ISP2532 (30 TB LUN)**  
### *Guia completo baseado em um caso real (CentOS 8.3 + SCST + FC 8Gb)*

Este documento mostra, passo a passo, como transformei um **Dell NX3200** em um **storage Fibre Channel completo**, expondo um **LUN de 30 TB via FC** usando:

- **SCST (open source)**  
- **QLogic ISP2532 (8Gb)** em modo *Target*  
- **CentOS 8.3 (Vault)**  
- Disco físico `/dev/sdb` (RAID PERC)

Todos os comandos abaixo foram extraídos e depurados a partir do **history real** da sessão — nada aqui é fictício.  
Isso significa que este guia **funciona exatamente como está**.

---

## **📌 Ambiente**
| Componente | Detalhes |
|-----------|----------|
| Servidor | Dell NX3200 |
| Fibre Channel | QLogic ISP2532 8Gb |
| Sistema | CentOS Linux release 8.3.2011 (Vault) |
| Kernel | 4.18.0-240.el8.x86_64 |
| Disco exportado como LUN | /dev/sdb (~30TB RAID PERC) |
| Framework Storage | SCST (qla2xxx_scst + qla2x00tgt) |

---

## **📍 1. Verificando hardware e estado inicial FC**

```bash
lspci | grep -i fibre
lsmod | grep qla2xxx

cat /sys/class/fc_host/host*/port_name
cat /sys/class/fc_host/host*/device/scsi_host/host*/active_mode
```

Saída esperada inicialmente:  
`active_mode = Initiator`

---

## **📍 2. Preparando repositórios CentOS 8.3 (Vault)**

```bash
sed -i 's|mirrorlist=|#mirrorlist=|g' /etc/yum.repos.d/CentOS-*.repo
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*.repo

yum clean all
yum makecache
yum update -y
```

---

## **📍 3. Limpando targetcli/LIO (para evitar conflito com SCST)**

```bash
systemctl stop target
systemctl disable target
yum remove -y targetcli
```

---

## **📍 4. Preparando /dev/sdb (30TB) para SCST**

> **Importante:** não monte `/dev/sdb` localmente quando for exportar via FC. 

```bash
wipefs -a /dev/sdb
sgdisk --zap-all /dev/sdb
mkfs.xfs /dev/sdb       # opcional para testes locais
umount /storage
```

---

## **📍 5. Instalando dependências de build**

```bash
yum install -y which bzip2 gcc kernel-devel elfutils-libelf-devel make \
perl perl-Data-Dumper perl-ExtUtils-MakeMaker rpm-build tar git
```

---

## **📍 6. Clonando e compilando SCST via RPMs**

```bash
cd /usr/src
git clone https://github.com/SCST-project/scst.git

cd scst
make 2release
make rpm
rpm -Uvh /usr/src/packages/RPMS/x86_64/*.rpm
```

---

## **📍 7. Recompilando módulos SCST com suporte a QLogic Target**

```bash
cd /usr/src/scst
make clean

BUILD_2X_MODULE=y CONFIG_SCSI_QLA_FC=y CONFIG_SCSI_QLA2XXX_TARGET=y make all
BUILD_2X_MODULE=y CONFIG_SCSI_QLA_FC=y CONFIG_SCSI_QLA2XXX_TARGET=y make install

depmod -a
```

Verificar módulos extras:

```bash
ls /lib/modules/$(uname -r)/extra | grep qla
```

---

## **📍 8. Carregando módulos SCST + QLogic Target**

```bash
rmmod qla2xxx        # remove driver iniciator padrão (IMPORTANTE)

modprobe scst
modprobe qla2xxx_scst
modprobe qla2x00tgt
modprobe scst_vdisk
```

Validar:

```bash
lsmod | grep -E 'qla|scst'
```

---

## **📍 9. Habilitar a porta Fibre Channel como TARGET**

```bash
ls -l /sys/kernel/scst_tgt/targets/qla2x00t/

echo "1" > /sys/kernel/scst_tgt/targets/qla2x00t/21:00:00:1b:32:94:31:e3/enabled

cat /sys/class/fc_host/host*/device/scsi_host/host*/active_mode
```

Resultado esperado:  
`Target`

---

## **📍 10. Criando o LUN (modo BLOCKIO) usando scstadmin**

### Abrir device:

```bash
scstadmin -open_dev lun0 -handler vdisk_blockio \
  -attributes filename=/dev/sdb,write_through=0
```

### Adicionar LUN 0 ao target:

```bash
scstadmin -add_lun 0 -driver qla2x00t \
  -target 21:00:00:1b:32:94:31:e3 -device lun0
```

### Validar:

```bash
scstadmin -list_device
scstadmin -list_target
```

---

## **📍 11. Gerar configuração persistente**

```bash
rm -rf /etc/scst.conf
scstadmin -write_config /etc/scst.conf
```

Ativar SCST no boot:

```bash
systemctl enable scst
systemctl restart scst
```

---

## **📍 12. Testando desempenho no Windows (DiskSpd)**

> **Nota:** o DiskSpd **NÃO** consegue abrir `\\.\PhysicalDriveX` em discos **maiores que 16 TB** (limitação da API RAW do Windows).  
> Portanto, teste com **arquivo grande** em um volume NTFS do LUN.

### Criar um volume simples NTFS e gerar um arquivo de 50 GB:

```powershell
fsutil file createnew G:\teste50GB.img 50000000000
```

### Teste de IOPS (random 4K, 50/50):

```cmd
diskspd.exe -b4k -d60 -o8 -t4 -r -w50 G:\teste50GB.img
```

### Teste de throughput (leitura sequencial 1MB):

```cmd
diskspd.exe -b1M -d30 -o4 -t2 -r -w0 G:\teste50GB.img
```

---

## **📍 13. Outras validações no TARGET Linux**

### Sessões FC:
```bash
scstadmin -list_sessions
```

### I/O no disco:
```bash
iostat -xm 2 | grep sdb
cat /proc/diskstats | grep sdb
```

---

## **📌 Resultado Final**
✔️ QLogic ISP2532 operando em **modo Target**  
✔️ SCST exportando **LUN 30TB via Fibre Channel**  
✔️ Windows reconhecendo como **SCST_BIO lun0**  
✔️ Desempenho sólido (FC 8Gb) medido via arquivo de teste  
✔️ Tudo feito com **open source** em hardware existente

---

## **🙌 Créditos & Notas**
- Projeto real executado em 2026.
- SCST + qla2x00tgt com CentOS 8.3/Kernel 4.18.0-240.
- Obrigado à comunidade SCST e aos inúmeros exemplos públicos que ajudam a viabilizar soluções enterprise com software livre.

---

## **📎 Quer contribuir?**
Pull requests são bem‑vindos!  
Sugestões de tópicos futuros: Multipath, ALUA, HA com Pacemaker/Corosync, replicação DRBD.

