# Intelecto

Agente de IA conectado ao WhatsApp via Evolution API, com LLM via OpenRouter.

```
WhatsApp â†’ Evolution API â†’ FastAPI (webhook) â†’ OpenRouter (LLM) â†’ resposta â†’ Evolution API â†’ WhatsApp
```

## PrÃ©-requisitos

- Docker e Docker Compose instalados
- Chave de API do [OpenRouter](https://openrouter.ai/)
- Um nÃºmero de WhatsApp para conectar ao bot

## InstalaÃ§Ã£o

### 1. Clonar o repositÃ³rio

```bash
git clone <url-do-repositorio>
cd intelecto-testes
```

### 2. Configurar variÃ¡veis de ambiente

Copie o arquivo de exemplo e preencha com suas credenciais:

```bash
cp .env.example .env
```

Edite o `.env`:

| VariÃ¡vel | DescriÃ§Ã£o | Exemplo |
|----------|-----------|---------|
| `OPENROUTER_API_KEY` | Chave da API do OpenRouter | `sk-or-v1-xxxx` |
| `OPENROUTER_MODEL` | Modelo LLM a utilizar | `anthropic/claude-sonnet-4-20250514` |
| `EVOLUTION_API_URL` | URL interna da Evolution API | `http://evolution-api:8080` |
| `EVOLUTION_API_KEY` | Chave de autenticaÃ§Ã£o da Evolution API | `minha-chave-secreta` |
| `EVOLUTION_INSTANCE_NAME` | Nome da instÃ¢ncia do WhatsApp | `intelecto` |
| `LOG_LEVEL` | NÃ­vel de log | `info` |

### 3. Personalidade do agente

O arquivo `soul.md` na raiz do projeto define a personalidade do agente. Edite conforme desejado antes de subir.

## Subindo o sistema

```bash
docker compose up --build
```

Isso inicia dois containers:

| Container | Porta | DescriÃ§Ã£o |
|-----------|-------|-----------|
| **intelecto** | 8000 | App FastAPI (webhook + API) |
| **evolution-api** | 8085 | Gateway WhatsApp (Evolution API v1.8.0) |

Para rodar em background:

```bash
docker compose up --build -d
```

Para reconstruir apenas o app apÃ³s mudanÃ§as no cÃ³digo:

```bash
docker compose up --build intelecto -d
```

## Conectando o WhatsApp

### 1. Criar a instÃ¢ncia na Evolution API

Acesse o painel da Evolution API em `http://localhost:8085` ou crie via API:

```bash
curl -X POST http://localhost:8085/instance/create \
  -H "apikey: SUA_EVOLUTION_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "instanceName": "intelecto",
    "qrcode": true
  }'
```

A resposta inclui um QR Code em base64. VocÃª tambÃ©m pode acessar o QR Code pelo painel web.

### 2. Escanear o QR Code

Abra o WhatsApp no celular:
1. VÃ¡ em **ConfiguraÃ§Ãµes > Aparelhos conectados**
2. Toque em **Conectar um aparelho**
3. Escaneie o QR Code gerado

### 3. Verificar conexÃ£o

```bash
curl -H "apikey: SUA_EVOLUTION_API_KEY" \
  http://localhost:8085/instance/connectionState/intelecto
```

O status deve ser `"open"` quando conectado.

### 4. Webhook (automÃ¡tico)

O Intelecto configura o webhook automaticamente ao iniciar. Ele verifica se a instÃ¢ncia existe e se o webhook estÃ¡ apontando para `http://intelecto:8000/webhook/whatsapp`. Se nÃ£o estiver, configura sozinho.

Caso precise configurar manualmente:

```bash
curl -X POST http://localhost:8085/webhook/set/intelecto \
  -H "apikey: SUA_EVOLUTION_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "enabled": true,
    "url": "http://intelecto:8000/webhook/whatsapp",
    "webhook_by_events": false,
    "events": ["MESSAGES_UPSERT"]
  }'
```

## Verificando se estÃ¡ tudo funcionando

### Status dos containers

```bash
docker compose ps
```

### Logs do Intelecto

```bash
docker compose logs -f intelecto
```

### Teste rÃ¡pido

Envie uma mensagem para o nÃºmero do WhatsApp conectado. Nos logs do Intelecto vocÃª deve ver:

```
INFO src.main: Mensagem de <remetente>: <texto>
INFO httpx: HTTP Request: POST https://openrouter.ai/api/v1/chat/completions "HTTP/1.1 200 OK"
INFO src.channels.whatsapp: Mensagem enviada para <destinatario>
```

## Desenvolvimento local (sem Docker)

```bash
pip install -r requirements.txt
uvicorn src.main:app --reload --port 8000
```

Neste caso, ajuste `EVOLUTION_API_URL` no `.env` para apontar para a URL acessÃ­vel da Evolution API (ex: `http://localhost:8085`).

## Estrutura do projeto

```
â”œâ”€â”€ docker-compose.yml    # OrquestraÃ§Ã£o dos containers
â”œâ”€â”€ Dockerfile            # Build do app Python
â”œâ”€â”€ .env                  # VariÃ¡veis de ambiente (nÃ£o commitado)
â”œâ”€â”€ .env.example          # Template das variÃ¡veis
â”œâ”€â”€ soul.md               # Personalidade do agente
â”œâ”€â”€ requirements.txt      # DependÃªncias Python
â”œâ”€â”€ data/                 # Dados persistentes (lid_map.json, etc)
â””â”€â”€ src/
    â”œâ”€â”€ main.py           # FastAPI app, webhook, lifespan
    â”œâ”€â”€ config.py         # ConfiguraÃ§Ãµes (pydantic-settings)
    â”œâ”€â”€ agent/
    â”‚   â”œâ”€â”€ core.py       # Orquestrador do agente
    â”‚   â””â”€â”€ soul.py       # Carrega soul.md
    â”œâ”€â”€ channels/
    â”‚   â””â”€â”€ whatsapp.py   # Cliente Evolution API (envio/recebimento)
    â””â”€â”€ llm/
        â”œâ”€â”€ base.py       # Interface abstrata do LLM
        â””â”€â”€ openrouter.py # ImplementaÃ§Ã£o OpenRouter
```

## Troubleshooting

### Mensagens nÃ£o chegam no bot
- Verifique se a instÃ¢ncia estÃ¡ conectada (`connectionState` = `open`)
- Verifique os logs: `docker compose logs -f intelecto`
- O webhook Ã© configurado automaticamente, mas confirme com:
  ```bash
  curl -H "apikey: SUA_EVOLUTION_API_KEY" http://localhost:8085/webhook/find/intelecto
  ```

### Bot recebe mas nÃ£o responde
- Verifique os logs por erros na chamada ao OpenRouter
- Confirme que `OPENROUTER_API_KEY` estÃ¡ vÃ¡lida no `.env`
- Verifique se o mapeamento LID existe em `data/lid_map.json`

### InstÃ¢ncia perdida apÃ³s restart
- A Evolution API persiste instÃ¢ncias nos volumes Docker (`evolution_instances`, `evolution_store`)
- Se os volumes forem removidos, Ã© necessÃ¡rio recriar a instÃ¢ncia e escanear o QR Code novamente
- O webhook serÃ¡ reconfigurado automaticamente pelo Intelecto

## Guia RÃ¡pido

```bash
# 1. Clonar e configurar
git clone git@github.com:inematds/intelecto-testes.git
cd intelecto-testes
cp .env.example .env
# Edite o .env com suas chaves

# 2. Subir os containers
docker compose up --build -d

# 3. Acessar o painel da Evolution API
# Abra http://localhost:8085 no navegador
# Crie a instÃ¢ncia com o nome "intelecto"
# Escaneie o QR Code no WhatsApp (ConfiguraÃ§Ãµes > Aparelhos conectados)

# 4. Pronto! Mande uma mensagem no WhatsApp e o bot responde.

# Comandos Ãºteis:
docker compose logs -f intelecto   # Ver logs
docker compose restart intelecto   # Reiniciar apÃ³s trocar .env
docker compose down                # Parar tudo
```

## PrÃ©-requisitos por sistema operacional

### Linux (Ubuntu/Debian)

```bash
# Atualizar pacotes
sudo apt update

# Instalar Docker
sudo apt install -y docker.io docker-compose-v2

# Adicionar seu usuÃ¡rio ao grupo docker (evita usar sudo)
sudo usermod -aG docker $USER
# FaÃ§a logout e login novamente para aplicar

# Instalar Git
sudo apt install -y git

# Verificar instalaÃ§Ã£o
docker --version
docker compose version
git --version
```

### Windows

1. **Docker Desktop** â€” Baixe e instale em [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)
   - Durante a instalaÃ§Ã£o, habilite o **WSL 2** quando solicitado
   - ApÃ³s instalar, abra o Docker Desktop e aguarde ele iniciar

2. **Git** â€” Baixe e instale em [git-scm.com](https://git-scm.com/)
   - Na instalaÃ§Ã£o, mantenha as opÃ§Ãµes padrÃ£o

3. **Terminal** â€” Use o **Git Bash** (instalado junto com o Git) ou o **PowerShell**

4. **Verificar instalaÃ§Ã£o** â€” Abra o terminal e rode:
   ```bash
   docker --version
   docker compose version
   git --version
   ```

ApÃ³s instalar os prÃ©-requisitos, siga o [Guia RÃ¡pido](#guia-rÃ¡pido) acima.

## Usando Ollama (LLM local) â€” Futuro

O Intelecto estÃ¡ preparado para suportar o [Ollama](https://ollama.com/) como provedor de LLM local, rodando modelos diretamente na sua mÃ¡quina sem depender de APIs externas.

### Requisitos de hardware

| Modelo | RAM mÃ­nima | GPU (opcional) | ObservaÃ§Ã£o |
|--------|-----------|----------------|------------|
| 7B (ex: llama3.1:8b) | 8 GB | 4 GB VRAM | Bom para testes |
| 13B | 16 GB | 8 GB VRAM | EquilÃ­brio qualidade/velocidade |
| 70B | 64 GB | 24+ GB VRAM | Melhor qualidade, exige hardware potente |

Com GPU NVIDIA a inferÃªncia Ã© significativamente mais rÃ¡pida. Sem GPU, roda na CPU (mais lento).

### Como vai funcionar

1. Adicionar o Ollama no `docker-compose.yml`:

```yaml
ollama:
  image: ollama/ollama
  ports:
    - "11434:11434"
  volumes:
    - ollama_data:/root/.ollama
  # Para usar GPU NVIDIA, descomente:
  # deploy:
  #   resources:
  #     reservations:
  #       devices:
  #         - driver: nvidia
  #           count: all
  #           capabilities: [gpu]
```

2. Baixar um modelo:

```bash
docker compose exec ollama ollama pull llama3.1:8b
```

3. Configurar no `.env`:

```env
LLM_PROVIDER=ollama
OLLAMA_URL=http://ollama:11434
OLLAMA_MODEL=llama3.1:8b
```

4. Reiniciar o Intelecto:

```bash
docker compose up --build -d
```

### DiferenÃ§as entre OpenRouter e Ollama

| | OpenRouter | Ollama |
|---|-----------|--------|
| **Onde roda** | Nuvem (API externa) | Local (sua mÃ¡quina) |
| **Custo** | Gratuito (limitado) ou pago | Gratuito (sem limites) |
| **Privacidade** | Dados passam por terceiros | Tudo local |
| **Velocidade** | Depende da internet | Depende do hardware |
| **Modelos** | Centenas disponÃ­veis | Precisa baixar cada um |
| **Setup** | SÃ³ precisa de API key | Precisa de hardware adequado |

---

Projeto adaptado da comunidade INEMA TDS - Marca Arckhe