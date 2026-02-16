# One API

_✨ Acesse todos os grandes modelos de linguagem através do formato padrão da API da OpenAI, pronto para uso ✨_

> **NOTA:** Este projeto é open source. Os usuários devem utilizá-lo em conformidade com os [Termos de Uso](https://openai.com/policies/terms-of-use) da OpenAI e com as leis e regulamentos aplicáveis. Não deve ser usado para fins ilegais.

> **AVISO:** Após o primeiro login com o usuário root, altere imediatamente a senha padrão `123456`!

---

## Funcionalidades

1. **Suporte a múltiplos modelos de IA:**
   - OpenAI ChatGPT (incluindo Azure OpenAI API)
   - Anthropic Claude (incluindo AWS Claude)
   - Google PaLM2/Gemini
   - Mistral
   - ByteDance Doubao (Volcano Engine)
   - Baidu ERNIE
   - Alibaba Qwen
   - iFlytek Spark
   - Zhipu ChatGLM
   - 360 AI Brain
   - Tencent Hunyuan
   - Moonshot AI
   - Baichuan
   - MINIMAX
   - Groq
   - Ollama
   - 01.AI (Yi)
   - StepFun
   - Coze
   - Cohere
   - DeepSeek
   - Cloudflare Workers AI
   - DeepL
   - together.ai
   - novita.ai
   - SiliconCloud
   - xAI

2. Suporte à configuração de mirrors e diversos [serviços proxy de terceiros](https://iamazing.cn/page/openai-api-third-party-services).

3. Suporte ao acesso a múltiplos canais via **balanceamento de carga**.

4. Suporte ao **modo stream** — transmissão em fluxo para efeito de digitação em tempo real.

5. Suporte a **deploy multi-servidor** (veja a seção [Deploy Multi-Servidor](#deploy-multi-servidor)).

6. Suporte ao **gerenciamento de tokens** — defina tempo de expiração, cota, faixa de IPs permitidos e modelos permitidos.

7. Suporte ao **gerenciamento de códigos de resgate** — geração em lote e exportação; use códigos para recarregar contas.

8. Suporte ao **gerenciamento de canais** — criação em lote de canais.

9. Suporte a **grupos de usuários** e **grupos de canais** — defina multiplicadores diferentes para cada grupo.

10. Suporte à **configuração de lista de modelos** por canal.

11. Suporte à **visualização detalhada de consumo de cota**.

12. Suporte a **recompensas por convite de usuários**.

13. Suporte à exibição de cotas em dólares americanos.

14. Suporte a publicação de avisos, configuração de link de recarga e cota inicial para novos usuários.

15. Suporte a **mapeamento de modelos** — redireciona o modelo solicitado pelo usuário. Se não for necessário, não configure, pois isso faz com que o corpo da requisição seja reconstruído ao invés de simplesmente repassado, e campos ainda não oficialmente suportados podem não ser transmitidos.

16. Suporte a **retry automático** em caso de falha.

17. Suporte à **interface de geração de imagens**.

18. Suporte ao [Cloudflare AI Gateway](https://developers.cloudflare.com/ai-gateway/providers/openai/) — preencha a parte de proxy nas configurações do canal com `https://gateway.ai.cloudflare.com/v1/ACCOUNT_TAG/GATEWAY/openai`.

19. Suporte a **personalizações avançadas:**
    - Personalizar nome do sistema, logo e rodapé.
    - Personalizar página inicial e página "Sobre" com HTML & Markdown ou via iframe.

20. Suporte à chamada da API de gerenciamento via token de acesso do sistema — **estenda e personalize** funcionalidades do One API sem necessidade de fork/modificação. Veja a [documentação da API](./docs/API.md).

21. Suporte ao Cloudflare Turnstile para verificação de usuários.

22. Suporte ao gerenciamento de usuários com **múltiplos métodos de login/registro:**
    - Login por e-mail (com whitelist de domínios) e redefinição de senha por e-mail.
    - Login via autorização Feishu (Lark).
    - Login via autorização GitHub.
    - Login via conta oficial do WeChat (requer deploy adicional do [WeChat Server](https://github.com/songquanpeng/wechat-server)).

23. Suporte a **troca de temas** — defina a variável de ambiente `THEME` (padrão: `default`).

24. Integração com [Message Pusher](https://github.com/songquanpeng/message-pusher) para enviar alertas a diversos apps.

---

## Deploy

### Deploy via Docker

```shell
# Deploy com SQLite:
docker run --name one-api -d --restart always -p 3000:3000 -e TZ=America/Sao_Paulo -v /home/ubuntu/data/one-api:/data justsong/one-api

# Deploy com MySQL (adicione o parâmetro SQL_DSN):
docker run --name one-api -d --restart always -p 3000:3000 -e SQL_DSN="root:123456@tcp(localhost:3306)/oneapi" -e TZ=America/Sao_Paulo -v /home/ubuntu/data/one-api:/data justsong/one-api
```

- A primeira porta `3000` em `-p 3000:3000` é a porta do host — modifique conforme necessário.
- Dados e logs serão salvos em `/home/ubuntu/data/one-api` no host. Certifique-se de que o diretório existe e tem permissão de escrita.
- Se falhar ao iniciar, adicione `--privileged=true`.
- Se não conseguir baixar a imagem, substitua `justsong/one-api` por `ghcr.io/songquanpeng/one-api`.
- Se tiver alta concorrência, **obrigatoriamente** configure `SQL_DSN` (veja Variáveis de Ambiente).

**Comando de atualização:**
```shell
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock containrrr/watchtower -cR
```

**Configuração de referência do Nginx:**
```nginx
server {
   server_name seudominio.com.br;  # Modifique para seu domínio

   location / {
          client_max_body_size  64m;
          proxy_http_version 1.1;
          proxy_pass http://localhost:3000;  # Modifique a porta se necessário
          proxy_set_header Host $host;
          proxy_set_header X-Forwarded-For $remote_addr;
          proxy_cache_bypass $http_upgrade;
          proxy_set_header Accept-Encoding gzip;
          proxy_read_timeout 300s;  # GPT-4 precisa de timeout mais longo
   }
}
```

**Configurar HTTPS com Let's Encrypt:**
```bash
# Ubuntu — instalar certbot:
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
# Gerar certificados e modificar configuração do Nginx
sudo certbot --nginx
# Seguir as instruções
# Reiniciar Nginx
sudo service nginx restart
```

**Conta inicial:** usuário `root`, senha `123456`.

### Deploy via Docker Compose

```shell
# Suporta MySQL, dados armazenados em ./data/mysql
docker-compose up -d

# Verificar status do deploy
docker-compose ps
```

### Deploy Manual

1. Baixe o executável dos [GitHub Releases](https://github.com/songquanpeng/one-api/releases/latest) ou compile a partir do código-fonte:

```shell
git clone https://github.com/songquanpeng/one-api.git

# Compilar frontend
cd one-api/web/default
npm install
npm run build

# Compilar backend
cd ../..
go mod download
go build -ldflags "-s -w" -o one-api
```

2. Executar:
```shell
chmod u+x one-api
./one-api --port 3000 --log-dir ./logs
```

3. Acesse [http://localhost:3000/](http://localhost:3000/) e faça login. Conta inicial: `root` / `123456`.

---

## Deploy Multi-Servidor

1. Todos os servidores devem ter o mesmo valor de `SESSION_SECRET`.
2. **Obrigatório** configurar `SQL_DSN` — use MySQL, não SQLite. Todos os servidores devem conectar ao mesmo banco de dados.
3. Todos os servidores secundários devem definir `NODE_TYPE` como `slave` (padrão é `master`).
4. Configure `SYNC_FREQUENCY` para sincronização periódica da configuração a partir do banco. Recomendado habilitar Redis, tanto em servidores primários quanto secundários.
5. Servidores secundários podem configurar `FRONTEND_BASE_URL` para redirecionar requisições de página ao servidor primário.
6. Instale Redis **separadamente** em cada servidor secundário. Configure `REDIS_CONN_STRING` para zero acesso ao banco enquanto o cache estiver válido, reduzindo latência.
7. Se o servidor primário também tiver alta latência ao banco, habilite Redis e configure `SYNC_FREQUENCY`.

---

## Uso

1. Na página **Canais**, adicione suas API Keys.
2. Na página **Tokens**, crie tokens de acesso.
3. Use os tokens para acessar o One API — o formato é idêntico à [API da OpenAI](https://platform.openai.com/docs/api-reference/introduction).

Configure o **API Base** para o endereço do seu deploy do One API, por exemplo: `https://seudominio.com.br`. A **API Key** é o token gerado no One API.

**Exemplo com a biblioteca oficial da OpenAI:**
```bash
OPENAI_API_KEY="sk-xxxxxx"
OPENAI_API_BASE="https://<HOST>:<PORT>/v1"
```

**Fluxo de funcionamento:**
```
Usuário → (usa key distribuída pelo One API) → One API → Encaminha para OpenAI / Azure / Claude / Gemini / etc.
```

Você pode especificar qual canal usar adicionando o ID do canal ao token: `Authorization: Bearer ONE_API_KEY-CHANNEL_ID` (apenas tokens criados por admin). Sem especificação, o balanceamento de carga distribui entre os canais disponíveis.

---

## Variáveis de Ambiente

O One API suporta leitura de variáveis de ambiente a partir de um arquivo `.env`. Renomeie `.env.example` para `.env`.

| # | Variável | Descrição | Exemplo |
|---|----------|-----------|---------|
| 1 | `REDIS_CONN_STRING` | Habilita Redis como cache. Se o banco tem baixa latência, não é necessário. | `redis://default:redispw@localhost:49153` |
| | `REDIS_PASSWORD` | Senha para cluster/sentinela Redis | |
| | `REDIS_MASTER_NAME` | Nome do nó master no modo sentinela | |
| 2 | `SESSION_SECRET` | Chave de sessão fixa — cookies permanecem válidos após reiniciar | `random_string` |
| 3 | `SQL_DSN` | Banco de dados (MySQL ou PostgreSQL) ao invés de SQLite. Crie o banco `oneapi` previamente. | MySQL: `root:123456@tcp(localhost:3306)/oneapi` / PostgreSQL: `postgres://postgres:123456@localhost:5432/oneapi` |
| | `SQL_MAX_IDLE_CONNS` | Máximo de conexões idle (padrão: 100) | |
| | `SQL_MAX_OPEN_CONNS` | Máximo de conexões abertas (padrão: 1000) | |
| | `SQL_CONN_MAX_LIFETIME` | Tempo de vida máximo da conexão em minutos (padrão: 60) | |
| 4 | `LOG_SQL_DSN` | Banco separado para a tabela `logs` | |
| 5 | `FRONTEND_BASE_URL` | Redireciona requisições de página (apenas servidores secundários) | `https://seudominio.com.br` |
| 6 | `MEMORY_CACHE_ENABLED` | Habilita cache em memória (pode causar atraso na atualização de cotas). Padrão: `false` | `true` |
| 7 | `SYNC_FREQUENCY` | Frequência de sincronização com o banco quando cache está habilitado, em segundos (padrão: 600) | `60` |
| 8 | `NODE_TYPE` | Tipo do nó: `master` (padrão) ou `slave` | `slave` |
| 9 | `CHANNEL_UPDATE_FREQUENCY` | Frequência de atualização do saldo dos canais, em minutos. Não definido = sem atualização. | `1440` |
| 10 | `CHANNEL_TEST_FREQUENCY` | Frequência de verificação dos canais, em minutos. | `1440` |
| 11 | `POLLING_INTERVAL` | Intervalo entre requisições ao atualizar/testar canais em lote, em segundos. Padrão: sem intervalo. | `5` |
| 12 | `BATCH_UPDATE_ENABLED` | Habilita agregação de atualizações em lote no banco (pode causar atraso). Padrão: `false`. Útil se tiver muitas conexões. | `true` |
| 13 | `BATCH_UPDATE_INTERVAL` | Intervalo da agregação em lote, em segundos (padrão: 5) | `5` |
| 14 | `GLOBAL_API_RATE_LIMIT` | Limite global de taxa de API (exceto relay), máx. requisições por IP em 3 minutos (padrão: 180) | |
| | `GLOBAL_WEB_RATE_LIMIT` | Limite global de taxa Web, máx. requisições por IP em 3 minutos (padrão: 60) | |
| 15 | `TIKTOKEN_CACHE_DIR` | Diretório de cache para codificação de tokens. Útil em ambientes offline. | |
| | `DATA_GYM_CACHE_DIR` | Mesma função que acima, mas com prioridade menor. | |
| 16 | `RELAY_TIMEOUT` | Timeout do relay em segundos. Padrão: sem timeout. | |
| 17 | `RELAY_PROXY` | Proxy para requisições à API | |
| 18 | `USER_CONTENT_REQUEST_TIMEOUT` | Timeout para download de conteúdo do usuário, em segundos | |
| 19 | `USER_CONTENT_REQUEST_PROXY` | Proxy para conteúdo do usuário (ex: imagens) | |
| 20 | `SQLITE_BUSY_TIMEOUT` | Timeout de lock do SQLite em milissegundos (padrão: 3000) | |
| 21 | `GEMINI_SAFETY_SETTING` | Configuração de segurança do Gemini (padrão: `BLOCK_NONE`) | |
| 22 | `GEMINI_VERSION` | Versão do Gemini usada (padrão: `v1`) | |
| 23 | `THEME` | Tema do sistema (padrão: `default`) | |
| 24 | `ENABLE_METRIC` | Desabilita canais com base na taxa de sucesso. Padrão: desligado. | `true` |
| 25 | `METRIC_QUEUE_SIZE` | Tamanho da fila de estatísticas de sucesso (padrão: 10) | |
| 26 | `METRIC_SUCCESS_RATE_THRESHOLD` | Limiar da taxa de sucesso (padrão: 0.8) | |
| 27 | `INITIAL_ROOT_TOKEN` | Se definido, cria automaticamente um token root na primeira inicialização | |
| 28 | `INITIAL_ROOT_ACCESS_TOKEN` | Se definido, cria automaticamente um token de administração do sistema na primeira inicialização | |
| 29 | `ENFORCE_INCLUDE_USAGE` | Força retorno de `usage` em modo stream. Padrão: desligado. | `true` |
| 30 | `TEST_PROMPT` | Prompt usado ao testar modelos | Padrão: `Print your model name exactly...` |

---

## Parâmetros de Linha de Comando

| Parâmetro | Descrição | Padrão |
|-----------|-----------|--------|
| `--port <número>` | Porta do servidor | `3000` |
| `--log-dir <diretório>` | Diretório dos logs | `./logs` |
| `--version` | Exibe a versão e sai | |
| `--help` | Exibe ajuda | |

---

## Perguntas Frequentes

1. **O que é cota e como é calculada?**
   - Cota = Multiplicador do Grupo × Multiplicador do Modelo × (tokens de prompt + tokens de completação × multiplicador de completação)
   - Multiplicador de completação: GPT-3.5 = 1.33, GPT-4 = 2 (consistente com a OpenAI)
   - Em modo não-stream, a API retorna o total de tokens consumidos, mas os multiplicadores de prompt e completação são diferentes.
   - Os multiplicadores padrão do One API já são os multiplicadores oficiais ajustados.

2. **Cota suficiente mas erro de "cota insuficiente"?**
   - Verifique a cota do **token** (separada da cota da conta). A cota do token é para o usuário definir um limite máximo de uso.

3. **Erro "nenhum canal disponível"?**
   - Verifique as configurações de grupo do usuário e grupo do canal, além das configurações de modelo do canal.

4. **Erro de teste do canal: `invalid character '<'...`?**
   - O retorno é HTML ao invés de JSON. Provavelmente o IP do deploy ou do proxy foi bloqueado pelo CloudFlare.

5. **ChatGPT Next Web erro: "Failed to fetch"?**
   - Não defina `BASE_URL` durante o deploy.
   - Verifique se o endereço da interface e a API Key estão corretos.
   - Verifique se HTTPS está habilitado (navegadores bloqueiam requisições HTTP em domínios HTTPS).

6. **Erro: "carga do grupo atual saturada, tente novamente mais tarde"?**
   - O canal upstream retornou erro 429 (rate limit).

7. **Meus dados serão perdidos após atualização?**
   - MySQL: Não.
   - SQLite: Precisa montar o volume conforme o comando de deploy para persistir o arquivo `one-api.db`. Sem isso, os dados serão perdidos ao reiniciar o container.

8. **Preciso alterar o banco antes de atualizar?**
   - Geralmente não. O sistema ajusta automaticamente durante a inicialização.

9. **Erro após modificar o banco manualmente: "consistência do banco comprometida"?**
   - Isso indica que a tabela `ability` tem registros com IDs de canal inexistentes. Provavelmente você deletou registros da tabela `channel` sem limpar os registros correspondentes na tabela `ability`.
   - Cada canal precisa ter um registro na tabela `ability` para cada modelo suportado.

---

## Projetos Relacionados

- [FastGPT](https://github.com/labring/FastGPT): Sistema de Q&A baseado em LLM com base de conhecimento
- [ChatGPT Next Web](https://github.com/Yidadaa/ChatGPT-Next-Web): Seu próprio ChatGPT multiplataforma com um clique
- [VChart](https://github.com/VisActor/VChart): Biblioteca de gráficos multiplataforma
- [VMind](https://github.com/VisActor/VMind): Solução inteligente de visualização open source
- [CherryStudio](https://github.com/CherryHQ/cherry-studio): Cliente de IA multiplataforma com suporte a múltiplos provedores e base de conhecimento local

---

## Observação

Este projeto usa a licença MIT. **Com base nisso**, é obrigatório manter a atribuição e o link para o projeto original no rodapé da página. Se não quiser manter a atribuição, obtenha autorização prévia. O mesmo se aplica a projetos derivados (forks).

De acordo com a licença MIT, os usuários assumem todos os riscos e responsabilidades pelo uso deste projeto.
