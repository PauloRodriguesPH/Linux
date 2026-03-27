## Exemplo de configuração do SCST (Generic SCSI Target Subsystem for Linux)

Este arquivo representa **a configuração do framework SCST**, responsável por expor
Logical Units (LUNs) a iniciadores Fibre Channel via driver QLogic (`qla2x00t`).

### Cenário descrito

- 4 LUNs baseadas em dispositivos LVM (`vdisk_blockio`)
- 1 adaptador Fibre Channel QLogic
- Apresentação **restrita por WWN** (zoning lógico via SCST)
- Cada host enxerga apenas a LUN autorizada
- Duas LUNs não são apresentadas a nenhum iniciador (reservadas)

---

## Runbook — configuração completa do SCST

> ✅ Este bloco contém **todo o arquivo de configuração (`/etc/scst.conf`)**
>  
> Use quando precisar **copiar, versionar ou restaurar** a configuração completa.

<details>
<summary><strong>➡️ Clique aqui para expandir e copiar o arquivo inteiro</strong></summary>

```conf
################################################################################
# SCST — Definição de dispositivos Block I/O (backend de storage)
#
# Cada DEVICE representa uma LUN baseada em um volume lógico (LVM).
# O SCST apenas exporta o bloco — não há filesystem envolvido.
################################################################################

HANDLER vdisk_blockio {

        # LUN01 — Volume lógico de 10 TB
        DEVICE lun01 {
                filename /dev/vg_stg/lun01
                size 10995116277760
        }

        # LUN02 — Volume lógico de 10 TB
        DEVICE lun02 {
                filename /dev/vg_stg/lun02
                size 10995116277760
        }

        # LUN03 — Volume lógico de 5 TB
        DEVICE lun03 {
                filename /dev/vg_stg/lun03
                size 5497558138880
        }

        # LUN04 — Volume lógico de 5 TB
        DEVICE lun04 {
                filename /dev/vg_stg/lun04
                size 5497558138880
        }
}


################################################################################
# COPY MANAGER
#
# Target interno do SCST usado para operações auxiliares
# (snapshots, cópias internas, gerenciamento de metadados).
# Não é exposto a hosts externos.
################################################################################

TARGET_DRIVER copy_manager {

        TARGET copy_manager_tgt {
                LUN 0 lun01
                LUN 1 lun02
                LUN 2 lun03
                LUN 3 lun04
        }
}


################################################################################
# TARGET FIBRE CHANNEL — QLogic (qla2x00t)
#
# Define o target FC visível no fabric, identificado pelo WWPN do host SCST.
################################################################################

TARGET_DRIVER qla2x00t {

        TARGET 21:00:00:1b:32:94:31:e3 {

                # Marca este target como baseado em hardware FC
                HW_TARGET

                # Habilita o target
                enabled 1

                # ID relativo do target no fabric
                rel_tgt_id 1


                ################################################################
                # LUNs globais definidas no target
                #
                # Estas LUNs são "conhecidas" pelo target, mas só ficam visíveis
                # para iniciadores se forem associadas a um GROUP.
                ################################################################

                # LUN03 e LUN04 NÃO são apresentadas a nenhum host
                # (reservadas ou ainda não liberadas)
                LUN 3 lun03
                LUN 4 lun04


                ################################################################
                # GROUP — HOST01
                #
                # Host autorizado a enxergar apenas a LUN01
                ################################################################

                GROUP HOST01_ONLY {

                        # LUN01 apresentada para este host
                        LUN 1 lun01

                        # WWPN do iniciador (host)
                        INITIATOR 21:00:00:24:ff:58:98:57
                }


                ################################################################
                # GROUP — HOST02
                #
                # Host autorizado a enxergar apenas a LUN02
                ################################################################

                GROUP HOST02_ONLY {

                        # LUN02 apresentada para este host
                        LUN 2 lun02

                        # WWPN do iniciador (host)
                        INITIATOR 10:00:00:00:c9:99:6f:c3
                }
        }
}
