# Configuração automática de pen drive NTFS no CasaOS via fstab

## O problema

Sem configuração, toda vez que o pen drive é conectado:

- O dispositivo muda de nome (`sdb1`, `sdc1`, etc.)
- As permissões de escrita não são aplicadas
- É necessário montar manualmente a cada conexão

A solução é configurar o `/etc/fstab` usando o **UUID** do pen drive, que é fixo e nunca muda.

---

## Passo 1 — Instalar o driver NTFS

```bash
sudo apt update
sudo apt install ntfs-3g -y
```

---

## Passo 2 — Descobrir o UUID do pen drive

Conecte o pen drive e rode:

```bash
lsblk
```

Identifique o dispositivo (ex: `sdc1`). Depois rode:

```bash
sudo blkid /dev/sdc1
```

Exemplo de saída:

```
/dev/sdc1: LABEL="New Volume" UUID="128C083A8C081B3D" TYPE="ntfs"
```

Anote o **UUID** (ex: `128C083A8C081B3D`).

---

## Passo 3 — Criar o ponto de montagem

```bash
sudo mkdir -p /mnt/pendrive
```

---

## Passo 4 — Descobrir seu UID e GID

```bash
id
```

Exemplo de saída:

```
uid=1000(yuan) gid=1000(yuan)
```

---

## Passo 5 — Editar o fstab

```bash
sudo nano /etc/fstab
```

Adicione essa linha no final (substitua o UUID se for diferente):

```
UUID=128C083A8C081B3D  /mnt/pendrive  ntfs-3g  rw,uid=1000,gid=1000,umask=0000,nofail,x-systemd.automount  0  0
```

Salve: `Ctrl+O` → `Enter` → `Ctrl+X`

### Explicação das opções

| Opção | Descrição |
|---|---|
| `rw` | Leitura e escrita |
| `uid=1000` | Dono dos arquivos (seu usuário) |
| `gid=1000` | Grupo do dono |
| `umask=0000` | Permissão total para leitura e escrita na interface |
| `nofail` | Sistema inicializa mesmo sem o pen drive conectado |
| `x-systemd.automount` | Monta automaticamente quando acessado |

---

## Passo 6 — Aplicar e testar

```bash
# Aplicar o fstab
sudo mount -a

# Ver os arquivos
ls /mnt/pendrive

# Testar escrita
touch /mnt/pendrive/teste.txt
ls /mnt/pendrive
```

Se o `teste.txt` aparecer, está tudo funcionando.

---

## Passo 7 — Tornar visível no CasaOS

Para o pen drive aparecer no painel de **Arquivos** do CasaOS, crie um link simbólico:

```bash
ln -s /mnt/pendrive /DATA/pendrive
```

---

## Ejetar corretamente antes de desconectar

**Sempre** rode esse comando antes de desconectar o pen drive para evitar perda de dados:

```bash
sudo sync && sudo umount /mnt/pendrive
```

Ou pelo CasaOS: clique com botão direito na pasta → **Ejetar**.

---

## Comportamento após configuração

| Situação | Resultado |
|---|---|
| Pen drive conectado | Monta automaticamente em `/mnt/pendrive` |
| Pen drive desconectado | Sistema inicializa normalmente |
| Reconectado | Monta novamente sem nenhum comando |
| Leitura e escrita | Funcionam direto na interface do CasaOS |
| Arquivo copiado | Visível no Windows após ejetar corretamente |

---

## Solução de problemas

### Pen drive montado como somente leitura

O Windows não ejetou corretamente. Rode:

```bash
sudo umount /dev/sdc1
sudo ntfsfix /dev/sdc1
sudo mount -a
```

### Erro "Input/output error"

O pen drive travou. Rode:

```bash
sudo umount -l /dev/sdc1
```

Desconecte e reconecte o pen drive, depois:

```bash
sudo mount -a
```

### Arquivo copiado no Linux não aparece no Windows

Você não ejetou corretamente. Sempre rode antes de desconectar:

```bash
sudo sync && sudo umount /mnt/pendrive
```

### Permissão negada na interface do CasaOS

```bash
sudo chmod 777 /mnt/pendrive
```

---

## Resumo dos comandos

```bash
# Instalar driver
sudo apt install ntfs-3g -y

# Ver UUID do pen drive
sudo blkid /dev/sdc1

# Criar ponto de montagem
sudo mkdir -p /mnt/pendrive

# Editar fstab
sudo nano /etc/fstab
# Adicionar: UUID=128C083A8C081B3D /mnt/pendrive ntfs-3g rw,uid=1000,gid=1000,umask=0000,nofail,x-systemd.automount 0 0

# Aplicar
sudo mount -a

# Ejetar com segurança
sudo sync && sudo umount /mnt/pendrive
```
