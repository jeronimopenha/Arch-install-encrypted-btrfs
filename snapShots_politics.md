# Política de snapshots para / (Notebook e PC)

| Onde                      | Hourly | Daily | Weekly | Monthly |
| ------------------------- | ------ | ----- | ------ | ------- |
| **Sistema (Timeshift)**   | 6      | 7     | 4      | 2       |
| **Home (local)**          | 12     | 7     | 4      | 0       |
| **Backup (local)**        | 24     | 14    | 8      | 0       |
| **HD Externo (frio)**     | 0      | 30    | 12     | 24      |

## Raiz 
- Timeshift + Timeshift autosnap


## HOME – SNAPSHOTS LOCAIS (BTRBK)

Aqui entra:
- Desktop
- Downloads
- Documentos
- (mas NÃO cache, Chrome, Steam etc)

**Objetivo**:

- Recuperação rápida
- Erro humano
- Exclusão acidental recente

### POLÍTICA LOCAL DA HOME (btrbk)
```
snapshot_preserve_hourly  12
snapshot_preserve_daily   7
snapshot_preserve_weekly  4
snapshot_preserve_monthly 0
```

### POLÍTICA LOCAL DO SUBVOL DE BACKUP
```
snapshot_preserve_hourly  24
snapshot_preserve_daily   14
snapshot_preserve_weekly  8
snapshot_preserve_monthly 0
```

### POLÍTICA HDS EXTERNOS – BACKUP REAL (BTRBK SEND)

Conservador de verdade, porque é frio e barato:
```
snapshot_preserve_hourly  0
snapshot_preserve_daily   30
snapshot_preserve_weekly  12
snapshot_preserve_monthly 24
```

# CRIAR O SUBVOLUME DOS DADOS DE BACKUP

1.1) Descobrir onde está a raiz real do BTRFS

```
sudo btrfs subvolume list /
```
Retorno esperado

```
ID 256 gen ... path @
ID 257 gen ... path @home
```

1.2) Montar a raiz real do BTRFS (se ainda não estiver)

```
sudo mkdir -p /mnt/btrfs-root
sudo mount -o subvolid=5 /dev/sdXn /mnt/btrfs-root
```

* PS: Trocar /dev/sdXn pelo dispositivo real (lsblk mostra).

1.3) Criar o subvolume de backup
```
sudo btrfs subvolume create /mnt/btrfs-root/@backup_dados
sudo btrfs subvolume create /mnt/btrfs-root/@backup_dados/live
sudo btrfs subvolume create /mnt/btrfs-root/@backup_dados/snapshots
```
Resultado:
```
@           → sistema
@home       → home
@backup_dados → dados importantes
@backup_dados/libve → dados vivos
@backup_dados/snapshots → snapshots e backups
```

## CRIAR O PONTO DE MONTAGEM

```
sudo mkdir -p /backup_dados
```

## MONTAR O SUBVOLUME NO SISTEMA

3.1) Montar manualmente para testar

```
sudo mount -o subvol=@backup_dados /dev/sdXn /backup_dados
```

Teste:

```
df -h | grep backup
```

3.2) Colocar no /etc/fstab (definitivo)

Editar:

```
sudo nano /etc/fstab
```

Adicionar uma linha assim:

```
UUID=XXXX-XXXX  /backup_dados  btrfs  subvol=@backup_dados,compress=zstd,noatime  0  0
```

Peguar o UUID com:

```
blkid
```

## MOVER AS PASTAS IMPORTANTES PRA LÁ

Exemplo:
```
mv ~/Documentos  /backup_dados/
mv ~/Imagens     /backup_dados/
mv ~/Vídeos      /backup_dados/
mv ~/Músicas     /backup_dados/
mv ~/Trabalhos   /backup_dados/
```

## CRIAR OS LINKS SIMBÓLICOS (PRA NÃO QUEBRAR NADA)
```
ln -s /backup_dados/Documentos  ~/Documentos
ln -s /backup_dados/Imagens     ~/Imagens
ln -s /backup_dados/Vídeos      ~/Vídeos
ln -s /backup_dados/Músicas     ~/Músicas
ln -s /backup_dados/Trabalhos   ~/Trabalhos
```

## SNAPSHOTS LOCAIS DESSE SUBVOLUME (BTRBK)

Esse subvolume será controlado pelo btrbk local.

Política :

```
# ==========================================
# LOG E DEFAULTS
# ==========================================
transaction_log        /var/log/btrbk.log

# mínimo que um snapshot fica intocado
snapshot_preserve_min  2d

# se quiser, dá pra deixar um preserve global bem genérico
# e sobrescrever por subvolume (como vou fazer abaixo)
# snapshot_preserve    24h 7d 4w

# por padrão, não forçar retenção mínima em targets
target_preserve_min    no

# ==========================================
# HOME (SSD) – snapshots CURTOS
# 12 hourly, 7 daily, 4 weekly, 0 monthly
# ==========================================
subvolume  /home
  snapshot_dir        /snapshots/home
  snapshot_preserve   12h 7d 4w

  # Se você quiser que a HOME também vá pro HD externo:
  target              /mnt/backup/home
  target_preserve     30d 12w 24m


# ==========================================
# BACKUP LOCAL DE DADOS
# (subvol onde estão Documentos, Música, Desktop etc.)
# 24 hourly, 14 daily, 8 weekly, 0 monthly
# ==========================================
subvolume  /home/jeronimo/backup_dados/live
  snapshot_dir        /snapshots/backup_dados
  snapshot_preserve   24h 14d 8w

  # Cópia "fria" de longo prazo no HD externo
  target              /mnt/backup/backup_dados
  target_preserve     30d 12w 24m

```

## BACKUP EM HD EXTERNO OCORRER

```

sudo btrbk -c /etc/btrbk.conf run

sudo systemctl enable --now btrbk.timer

```


Criará os snapshots se precisar

Envia incrementalmente

Não duplica nada

Não quebra nada se o HD não estiver montado antes

