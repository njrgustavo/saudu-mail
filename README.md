Saudu Mailserver (Docker)

Este repositório contém um `compose.yaml` para subir o Docker Mailserver com o domínio `saudu.com.br`.

Como funciona
- Baseado na configuração mínima recomendada do Docker Mailserver [0].
- Expõe as portas SMTP/Submission/IMAPS (`25`, `465`, `587`, `993`).
- Usa certificados TLS de Let's Encrypt via `SSL_TYPE=letsencrypt` (monta `/etc/letsencrypt`).
- Ativa antispam (Rspamd), antivírus (ClamAV) e Fail2Ban.

Pré-requisitos na VPS
- Docker e Docker Compose instalados.
- DNS apontado: `A` para `mail.saudu.com.br` → IP da VPS; `MX` do domínio → `mail.saudu.com.br`.
- Firewall liberando portas `25`, `465`, `587`, `993`.
- (Recomendado) PTR/reverse DNS do IP apontando para `mail.saudu.com.br`.
- Certificados Let's Encrypt emitidos para `mail.saudu.com.br` (ex.: via `certbot`).

Subir o serviço
1) Copie os arquivos para a VPS.
2) Garanta que `/etc/letsencrypt` na VPS possui o certificado do host `mail.saudu.com.br`.
3) Execute:
   ```
   docker compose up -d
   ```

Criar contas de e-mail
- Adicione usuários:
  ```
  docker exec -it mailserver setup email add usuario@saudu.com.br SenhaForteAqui
  ```
- Liste contas:
  ```
  docker exec -it mailserver setup email list
  ```

DKIM/SPF/DMARC (melhor entrega)
- Gere DKIM no container:
  ```
  docker exec -it mailserver setup config dkim
  ```
- Publique registros DNS do domínio:
  - SPF: `TXT @  v=spf1 mx -all`
  - DKIM: `TXT selector._domainkey` com o valor gerado acima.
  - DMARC: `TXT _dmarc  v=DMARC1; p=quarantine; rua=mailto:postmaster@saudu.com.br`

Observações
- Altere `TZ` em `compose.yaml` se necessário.
- O caminho `./docker-data/dms/...` será criado automaticamente ao subir o serviço e mantém os dados persistentes.

[0] Tutorial oficial com exemplo básico: https://docker-mailserver.github.io/docker-mailserver/latest/examples/tutorials/basic-installation/#a-basic-example-with-relevant-environmental-variables

Roundcube Webmail
- Acesso local: `http://<IP_da_VPS>:8081/`
- Configuração no compose: `roundcube` usa SQLite (persistência em `./docker-data/roundcube/db/`) e está apontado para o IMAP/SMTP de `mail.saudu.com.br` (IMAP SSL `993`, SMTP STARTTLS `587`).
- Login: use os usuários criados no mailserver (ex.: `contato@saudu.com.br`).
- Produção com HTTPS:
  - Crie `A` para `webmail.saudu.com.br` → IP da VPS.
  - Opcional: configure Nginx/Apache como reverse proxy para encaminhar `webmail.saudu.com.br` → `http://127.0.0.1:8081/` e emita TLS com Let's Encrypt.
  - Alternativamente, mapeie outra porta externa e use um proxy existente.

Subir Roundcube
- Após editar o compose: `docker compose up -d`
- Logs: `docker logs roundcube --tail 200`