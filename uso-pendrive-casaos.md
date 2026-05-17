# Uso do pen drive exFAT no CasaOS

## Fluxo básico

Sempre siga essa ordem para evitar perda de arquivos:

```
Conectar → Montar → Copiar arquivos → Ejetar → Desconectar
```

---

## 1. Conectar o pen drive

Conecte o pen drive na porta USB e verifique se o sistema reconheceu:

```bash
lsblk -f
```

Procure o dispositivo exFAT (ex: `sdc1`). O nome pode mudar a cada conexão (`sdb1`, `sdc1`, `sdd1`).

---

## 2. Montar

```bash
udisksctl mount -b /dev/sdc1
```

Substitua `sdc1` pelo nome correto que apareceu no `lsblk`.

Exemplo de saída:

```
Mounted /dev/sdc1 at /run/media/yuan/PenDrive
```

Anote o caminho onde foi montado (ex: `/run/media/yuan/PenDrive`).

---

## 3. Copiar arquivos

**Pelo terminal:**

```bash
# Copiar arquivo para o pen drive
cp /caminho/do/arquivo /run/media/yuan/PenDrive/

# Copiar pasta inteira para o pen drive
cp -r /caminho/da/pasta /run/media/yuan/PenDrive/

# Ver arquivos no pen drive
ls /run/media/yuan/PenDrive
```

**Pela interface do CasaOS:**

O pen drive aparecerá automaticamente no painel de **Arquivos** após montar. Arraste e solte os arquivos normalmente.

---

## 4. Ejetar antes de desconectar

**OBRIGATÓRIO** — nunca desconecte sem ejetar, ou os arquivos não serão gravados:

```bash
sync && udisksctl unmount -b /dev/sdc1
```

- `sync` — força gravar tudo que está em cache no pen drive
- `udisksctl unmount` — desmonta com segurança

Após esse comando, pode desconectar o pen drive com segurança.

---

## Atalho rápido (recomendado)

Configure um atalho para não precisar digitar o comando completo:

```bash
echo "alias ejetar='sync && udisksctl unmount -b /dev/sdc1'" >> ~/.bashrc
source ~/.bashrc
```

Agora basta digitar:

```bash
ejetar
```

---

## Resumo do dia a dia

| Ação | Comando |
|---|---|
| Ver pen drive conectado | `lsblk -f` |
| Montar | `udisksctl mount -b /dev/sdc1` |
| Ver arquivos | `ls /run/media/yuan/PenDrive` |
| Ejetar | `sync && udisksctl unmount -b /dev/sdc1` |

---

## Por que preciso ejetar?

O Linux guarda os dados em memória RAM (cache) e grava no pen drive em segundo plano para ser mais rápido. Se desconectar sem ejetar, os dados que ainda estão no cache são perdidos e o arquivo não aparece no Windows.

O `sync` força gravar tudo antes de desmontar — equivale ao "Ejetar com segurança" do Windows.

---

## Problemas comuns

### Arquivo copiado não aparece no Windows

Você não ejetou corretamente. Sempre rode antes de desconectar:

```bash
sync && udisksctl unmount -b /dev/sdc1
```

### Pen drive não monta

```bash
lsblk -f
```

Verifique o nome correto do dispositivo e tente novamente com o nome atualizado.

### Erro de permissão ao copiar

```bash
sudo chmod 777 /run/media/yuan/PenDrive
```

### Ver qual dispositivo é o pen drive

O disco principal sempre é `sda`. Tudo que aparecer como `sdb`, `sdc`, `sdd` é dispositivo externo (pen drive, HD externo).
