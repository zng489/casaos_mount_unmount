# Guia: Recuperar Acesso ao CasaOS

## Pré-requisitos

- Acesso ao terminal ou SSH do servidor
- Usuário com permissão `sudo`

---

## Passo 1 — Instalar o sqlite3

```bash
sudo apt install sqlite3 -y
```

---

## Passo 2 — Verificar o usuário no banco de dados

```bash
sqlite3 /var/lib/casaos/db/user.db ".tables"
sqlite3 /var/lib/casaos/db/user.db "SELECT * FROM o_users;"
```

Exemplo de saída:
```
1|yuan|97c2c7955969bf6c082ebe412650d16c|admin|||||2026-04-20 18:25:20|
```

Campos: `id | username | password (MD5) | role | ...`

---

## Passo 3 — Redefinir a senha

Gere um novo hash MD5 e atualize o banco com `sudo`:

```bash
HASH=$(echo -n "sua_nova_senha" | md5sum | awk '{print $1}')
sudo sqlite3 /var/lib/casaos/db/user.db "UPDATE o_users SET password='$HASH' WHERE username='seu_usuario';"
```

> **Exemplo** com usuário `yuan` e senha `casaos123`:
> ```bash
> HASH=$(echo -n "casaos123" | md5sum | awk '{print $1}')
> sudo sqlite3 /var/lib/casaos/db/user.db "UPDATE o_users SET password='$HASH' WHERE username='yuan';"
> ```

---

## Passo 4 — Reiniciar o serviço de usuários

```bash
sudo systemctl restart casaos-user-service
```

---

## Passo 5 — Acessar o CasaOS

Descubra o IP do servidor:

```bash
ip a | grep inet
```

Acesse no navegador:

```
http://IP-DO-SERVIDOR:81
```

Faça login com o usuário e a nova senha definidos no Passo 3.

---

## Solução de Problemas

| Erro | Causa | Solução |
|------|-------|---------|
| `attempt to write a readonly database` | Falta de permissão | Use `sudo` antes do `sqlite3` |
| `Command 'sqlite3' not found` | Pacote não instalado | Execute o Passo 1 |
| `no such table: o_users` | Versão diferente do CasaOS | Tente `.tables` para listar as tabelas disponíveis |
| Página não carrega | Serviço parado | `sudo systemctl start casaos` |

---

## Notas de Segurança

- Após recuperar o acesso, defina uma senha forte e única.
- O CasaOS armazena senhas em **MD5**, que é um hash fraco. Evite reutilizar senhas de outros serviços.
- Restrinja o acesso à porta `81` ao uso local ou via VPN sempre que possível.
