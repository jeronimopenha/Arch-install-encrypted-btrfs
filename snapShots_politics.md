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
```
Resultado:
```
@           → sistema
@home       → home
@backup_dados → dados importantes
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

Política  (relembrando):

```
Hourly:  24
Daily:   14
Weekly:  8
Monthly: 0 (só local)
```

## BACKUP EM HD EXTERNO OCORRER

```

sudo btrbk -c /etc/btrbk.conf run
```


Criará os snapshots se precisar

Envia incrementalmente

Não duplica nada

Não quebra nada se o HD não estiver montado antes

