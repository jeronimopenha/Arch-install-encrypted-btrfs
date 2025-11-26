# Arquitetura Oficial do Servidor — BTRFS RAID1 + Snapshots + Backup Externo

Autor: Jeronimo  
Data: 2025  
Objetivo: Arquitetura definitiva de armazenamento, snapshots e backup para servidor doméstico com alta integridade, recuperação e migração fácil.

---

## 1. 🎯 OBJETIVO DA ARQUITETURA

Garantir:

- ✅ Proteção contra falha de disco
- ✅ Proteção contra apagamentos acidentais
- ✅ Proteção contra corrupção silenciosa
- ✅ Proteção contra ransomware
- ✅ Backup externo independente
- ✅ Migração rápida para outro servidor
- ✅ Separação clara entre:
  - sistema
  - dados críticos
  - serviços
  - mídia

---

## 2. 💽 VISÃO GERAL DOS DISPOSITIVOS

### 🔷 Discos Principais (Produção)

- 1× SSD 500 GB  
- 1× HD 500 GB (ou maior)

➡️ Formam juntos o **RAID1 BTRFS principal do servidor**.

---

### 🔶 Discos Secundários (Mídia)

- Vários HDs grandes
- Gerenciados por:
  - SnapRAID (paridade)
  - MergerFS (opcional)

➡️ Usados para:
- Jellyfin
- Filmes
- Séries
- ISOs
- Arquivos recriáveis

---

### 🔷 Disco Externo (Backup)

- 1× HD Externo
- Sistema de arquivos: **BTRFS**
- Usado apenas para:
  - Snapshots mensais
  - Dumps de banco
  - Dados críticos

---

## 3. 🧱 RAID1 BTRFS — DISCO PRINCIPAL

### ✅ Regra Fundamental

- ✅ RAID1 para **tudo que é crítico**
- ❌ RAID1 **não é usado para mídia**

---

### 🔧 Criação do RAID1 (Exemplo)

```bash
mkfs.btrfs -m raid1 -d raid1 /dev/sdSSD /dev/sdHD
mount /dev/sdSSD /mnt

4. 📂 LAYOUT DE SUBVOLUMES (OBRIGATÓRIO)

Dentro do RAID1:

/
├── @system        → Sistema operacional
├── @containers    → Docker / Podman
├── @db            → MariaDB / PostgreSQL
├── @nextcloud     → Dados do Nextcloud
└── @photoprism    → Fotos do PhotoPrism

/etc/fstab (Modelo)
UUID=XXXX  /                     btrfs  subvol=@system,compress=zstd,noatime  0 0
UUID=XXXX  /var/lib/containers   btrfs  subvol=@containers,compress=zstd,noatime  0 0
UUID=XXXX  /var/lib/mysql        btrfs  subvol=@db,compress=zstd,noatime  0 0
UUID=XXXX  /srv/nextcloud        btrfs  subvol=@nextcloud,compress=zstd,noatime  0 0
UUID=XXXX  /srv/photoprism       btrfs  subvol=@photoprism,compress=zstd,noatime  0 0

5. 🔧 ONDE FICA CADA SERVIÇO
Serviço	Subvolume
Sistema Linux	@system
Containers	@containers
Banco de Dados	@db
Nextcloud	@nextcloud
PhotoPrism	@photoprism
Jellyfin / Mídia	HDs SnapRAID
6. 🕒 POLÍTICA DE SNAPSHOTS
🔵 @system

Diário: 7

Semanal: 4

Mensal: 6

🟢 @containers

Diário: 5

Semanal: 2

Mensal: 3

🔶 @db

⚠️ Banco NÃO é snapshotado diretamente em produção.

Fluxo correto:

mysqldump nextcloud | zstd > /backups/db_nextcloud_$(date +%F).sql.zst


Depois snapshot da pasta.

Retenção:

Diário: 3

Semanal: 2

Mensal: 6

🔴 @nextcloud

Diário: 3–5

Semanal: 2

Mensal: 12 ✅

Procedimento:

Ativar manutenção:

sudo -u www-data php /var/www/nextcloud/occ maintenance:mode --on


Dump do banco

Snapshot do subvolume

Desativar manutenção

🔴 @photoprism

Diário: 1–3

Semanal: 2

Mensal: 12 ✅

7. ☁️ BACKUP PARA HD EXTERNO (BTRFS)
✅ O QUE VAI PARA O EXTERNO
Subvolume	Vai?
@nextcloud	✅
@photoprism	✅
Banco	✅
@system	Opcional
@containers	Opcional
✅ Método Correto (SEM rsync)

Primeiro envio:

btrfs send /snapshots/nextcloud_2025-03 |
btrfs receive /mnt/externo/nextcloud


Envios seguintes (incremental):

btrfs send -p \
  /snapshots/nextcloud_2025-03 \
  /snapshots/nextcloud_2025-04 | \
  btrfs receive /mnt/externo/nextcloud


Banco:

rsync /backups/db_nextcloud_2025-03.sql.zst /mnt/externo/db/

8. 🎥 SNAPRAID (APENAS PARA MÍDIA)

Usado apenas para:

Jellyfin

Filmes

Séries

ISOs

❌ Nunca usado para:

Nextcloud

Fotos

Bancos

Backups

9. 🔥 FLUXO DE RECUPERAÇÃO
✅ Falha de disco:

Substitui disco

Recria RAID1

btrfs receive do externo

Restaura banco

✅ Apagamento acidental:

Restaura snapshot local

✅ Ransomware:

Restaura snapshot

Externo intacto

✅ Migração para outro servidor:

Cria RAID1

btrfs receive do externo

Sobe containers

10. ❓ SSD + HD EM RAID1 FUNCIONA?

✅ Sim, funciona:

SSD + HD = RAID1 BTRFS

Limitações:

Velocidade limitada ao HD

Tamanho = menor disco

Durabilidade diferente

✅ Modelo recomendado inicialmente:
SSD 500GB + HD 500GB → RAID1 BTRFS

✅ Modelo ideal futuro:
2× SSD → RAID1 principal
HDs → mídia (SnapRAID)

11. ✅ CONCLUSÃO FINAL

Com essa arquitetura você tem:

✅ RAID1 BTRFS para dados críticos

✅ Snapshots por tipo de dado

✅ Backup mensal isolado

✅ Nextcloud e PhotoPrism totalmente protegidos

✅ Banco em SSD com desgaste irrelevante

✅ SnapRAID só para mídia

✅ Migração rápida e segura

📌 ESTE DOCUMENTO É A REFERÊNCIA OFICIAL DA SUA ESTRUTURA DE SERVIDOR.


---

Se você quiser, no próximo passo eu posso:

✅ Te entregar também um `README.md` de **operação diária/mensal**  
✅ Ou os **scripts automáticos** em `.sh` para:
- snapshot
- dump de banco
- envio com `btrfs send -p`
- limpeza automática

Se quiser, é só falar:
> “quero os scripts em .sh”
