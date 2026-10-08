# Tutorial: instalar o site do MIG no cPanel da Namecheap (tudo no cPanel)

Neste método **não instalas nada no teu PC**. Envias o projeto para o cPanel e o próprio cPanel instala e compila o site.

> **Aviso honesto:** não consegui testar a compilação completa no meu ambiente (sem internet). O servidor e o login de administrador foram testados, mas é possível que apareça algum erro no passo 5 ou 6. Se isso acontecer, copia a mensagem e envia-ma. Além disso, alguns alojamentos partilhados limitam a memória e podem interromper a compilação: a secção «Se a compilação falhar» no fim explica o que fazer.

---

## Passo 1 — Criar a base de dados

1. No cPanel, escreve **PostgreSQL** na pesquisa e abre **PostgreSQL Databases**.
2. Em *Create New Database*, escreve `mig` e cria. O cPanel acrescenta um prefixo (o nome final fica parecido com `migcaxxu_mig`).
3. Em *Add New User*, cria um utilizador (por exemplo `migapp`) com uma palavra-passe **só com letras e números**. Guarda-a.
4. Em *Add User To Database*, escolhe o utilizador e a base de dados, clica em Add e marca **ALL PRIVILEGES**.
5. Monta o teu endereço de ligação, trocando pelos teus nomes reais (vais precisar dele no passo 4):

```
postgresql://migcaxxu_migapp:A_TUA_PALAVRA_PASSE@localhost:5432/migcaxxu_mig
```

As tabelas são criadas sozinhas quando o site arranca pela primeira vez.

> Se não encontrares a ferramenta **PostgreSQL Databases**, para aqui e diz-me, porque o alojamento pode não incluir PostgreSQL e eu teria de adaptar a base de dados.

---

## Passo 2 — Enviar o projeto

1. No cPanel abre o **File Manager**.
2. Fica na pasta principal `/home/migcaxxu` (fora de `public_html`) e clica em **+ Folder**. Chama-lhe `migsite`.
3. Entra em `migsite`, clica em **Upload** e envia o ficheiro `mig-cpanel-projeto.zip` (o zip, sem o extrair).
4. Volta ao File Manager, clica com o botão direito no zip → **Extract**.
5. Confirma que dentro de `migsite` estão diretamente `package.json`, `server`, `src`, `public`, `db` e `dist` (e não uma pasta extra pelo meio). Depois podes apagar o zip.

---

## Passo 3 — Criar a aplicação Node.js

1. No cPanel abre **Setup Node.js App** e clica em **Create Application**.
2. Preenche:
   - **Node.js version:** a mais recente que apareça (22 se existir; serve qualquer uma a partir da 18)
   - **Application mode:** Production
   - **Application root:** `migsite`
   - **Application URL:** escolhe o teu domínio (`migcs.site`) e deixa o resto vazio
   - **Application startup file:** `dist/app.cjs`
3. Clica em **Create**.

---

## Passo 4 — Variáveis de ambiente

Na mesma página, em **Environment variables**, clica em *Add Variable* e cria cada uma:

| Nome | Valor |
|---|---|
| `DATABASE_URL` | o endereço do passo 1 |
| `ADMIN_PASSWORD` | a palavra-passe que queres para entrar em `/admin` |
| `SESSION_SECRET` | uma sequência aleatória com pelo menos 24 caracteres (por exemplo `k8Xp2mQ9vLw4Zr7TnB3cYd6F`) |
| `SITE_URL` | `https://migcs.site` |

Opcionais, só se quiseres os vídeos do TikTok no site (copia os valores do painel da Netlify, em *Site configuration → Environment variables*): `TIKTOK_CLIENT_KEY`, `TIKTOK_CLIENT_SECRET`, `TIKTOK_TOKEN_ENCRYPTION_KEY`, `TIKTOK_REFRESH_TOKEN`.

Clica em **Save**.

---

## Passo 5 — Instalar as dependências

Na página da aplicação clica em **Run NPM Install** e espera. Pode demorar alguns minutos. Quando terminar, aparece uma mensagem de sucesso.

---

## Passo 6 — Compilar o site

1. Na mesma página, clica em **Run JS script**.
2. Escolhe o script **`build:site`** e executa.
3. Espera uns minutos. Quando terminar, abre o File Manager em `migsite/dist`: deve existir uma pasta **`client`** e o ficheiro `app.cjs` deve ter ficado maior (antes tinha só uma linha).

---

## Passo 7 — Arrancar o site

Na página da aplicação clica em **Restart**. Depois abre o teu domínio.

---

## Passo 8 — Ativar o SSL (cadeado https)

1. No cPanel abre **SSL/TLS Status**.
2. Seleciona o teu domínio e clica em **Run AutoSSL**. Pode demorar alguns minutos.

Isto é importante: o login em `/admin` só funciona em `https`.

---

## Passo 9 — Testar

1. Abre o domínio: o site deve aparecer.
2. Abre `https://migcs.site/admin`, entra com a `ADMIN_PASSWORD`, muda um texto e guarda.
3. Envia um formulário de teste (por exemplo, uma parceria) e confirma que aparece a mensagem de sucesso.
4. Para ver os pedidos recebidos, usa o **phpPgAdmin** (na secção PostgreSQL do cPanel, se existir) e abre as tabelas `inventory_requests`, `skin_offers` e `partnership_requests`.

**Se aparecer uma página de erro:** abre o File Manager, entra em `migsite` e procura o ficheiro `stderr.log` (se existir). Copia as últimas linhas e envia-mas.

---

## Se a compilação falhar (passo 5 ou 6)

Os alojamentos partilhados têm limites de memória e processos, e a compilação do site é pesada. Se o botão não terminar, der erro ou a pasta `dist/client` não aparecer:

1. Tenta de novo uma vez (por vezes é só falta de recursos naquele momento).
2. Se continuar a falhar, abre **Manage Shell** (na secção «Exclusive for Namecheap Customers»), ativa o acesso e abre o terminal. Na página **Setup Node.js App** aparece, no topo da aplicação, uma linha «Enter to the virtual environment»: copia-a para o terminal, depois escreve `cd ~/migsite` e `npm run build:site`. Assim vês a mensagem de erro completa e podes enviar-ma.
3. Se o servidor não aguentar mesmo, há uma alternativa: compilar o site noutro sítio e enviar só o resultado já pronto. Diz-me e preparo-te esse caminho.

---

## Se o domínio ainda aponta para a Netlify

Se o domínio está registado na Namecheap e este alojamento está na mesma conta, normalmente já aponta para o sítio certo. Se não, em *Domain List → Manage → Nameservers* escolhe **Namecheap BasicDNS** e depois cria um registo **A Record** (`@`) com o IP partilhado que aparece no cPanel («Shared IP Address»). As alterações de DNS podem demorar até 24 horas.

---

## Para atualizar o site no futuro

Substitui os ficheiros alterados no File Manager, volta ao **Run JS script → `build:site`** (passo 6) e clica em **Restart**. Os textos editados em `/admin` ficam guardados na base de dados e não se perdem.

---

## O que mudou em relação à versão da Netlify

- **Login de administrador:** deixou de usar o Netlify Identity. Agora é só uma palavra-passe (a variável `ADMIN_PASSWORD`).
- **Dados antigos:** os pedidos já recebidos e os textos editados na Netlify **não passam sozinhos** para a nova base de dados. O site novo começa limpo, com os textos padrão. Se precisares desses dados, guarda-os antes de desligares a Netlify.
- **Funções e base de dados:** o que antes eram funções da Netlify agora corre no servidor Node.js da pasta `server/`, e a base de dados é o PostgreSQL do teu cPanel.
- **Link «Vender skins»** e o texto de privacidade já não mencionam a Netlify.
