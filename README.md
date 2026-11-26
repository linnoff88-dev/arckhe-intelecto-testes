ï»¿# Intelecto

Agente de IA conectado ao WhatsApp via Evolution API, com LLM via OpenRouter.

```
WhatsApp ê¢â â Evolution API ê¢â â FastAPI (webhook) ê¢â â OpenRouter (LLM) ê¢â â resposta ê¢â â Evolution API ê¢â â WhatsApp
```

## PrêÂ©-requisitos

- Docker e Docker Compose instalados
- Chave de API do [OpenRouter](https://openrouter.ai/)
- Um nêÂºmero de WhatsApp para conectar ao bot

## InstalaêÂ§êÂ£o

### 1. Clonar o repositêÂ³rio

```bash
git clone <url-do-repositorio>
cd intelecto-testes
```

### 2. Configurar variêÂ¡veis de ambiente

Copie o arquivo de exemplo e preencha com suas credenciais:

```bash
cp .env.example .env
```

Edite o `.env`:

| VariêÂ¡vel | DescriêÂ§êÂ£o | Exemplo |
|----------|-----------|---------|
| `OPENROUTER_API_KEY` | Chave da API do OpenRouter | `sk-or-v1-xxxx` |
| `OPENROUTER_MODEL` | Modelo LLM a utilizar | `anthropic/claude-sonnet-4-20250514` |
| `EVOLUTION_API_URL` | URL interna da Evolution API | `http://evolution-api:8080` |
| `EVOLUTION_API_KEY` | Chave de autenticaêÂ§êÂ£o da Evolution API | `minha-chave-secreta` |
| `EVOLUTION_INSTANCE_NAME` | Nome da instêÂ¢ncia do WhatsApp | `intelecto` |
| `LOG_LEVEL` | NêÂ­vel de log | `info` |

### 3. Personalidade do agente

O arquivo `soul.md` na raiz do projeto define a personalidade do agente. Edite conforme desejado antes de subir.

## Subindo o sistema

```bash
docker compose up --build
```

Isso inicia dois containers:

| Container | Porta | DescriêÂ§êÂ£o |
|-----------|-------|-----------|
| **intelecto** | 8000 | App FastAPI (webhook + API) |
| **evolution-api** | 8085 | Gateway WhatsApp (Evolution API v1.8.0) |

Para rodar em background:

```bash
docker compose up --build -d
```

Para reconstruir apenas o app apêÂ³s mudanêÂ§as no cêÂ³digo:

```bash
docker compose up --build intelecto -d
```

## Conectando o WhatsApp

### 1. Criar a instêÂ¢ncia na Evolution API

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

A resposta inclui um QR Code em base64. VocêÂª tambêÂ©m pode acessar o QR Code pelo painel web.

### 2. Escanear o QR Code

Abra o WhatsApp no celular:
1. VêÂ¡ em **ConfiguraêÂ§êÂµes > Aparelhos conectados**
2. Toque em **Conectar um aparelho**
3. Escaneie o QR Code gerado

### 3. Verificar conexêÂ£o

```bash
curl -H "apikey: SUA_EVOLUTION_API_KEY" \
  http://localhost:8085/instance/connectionState/intelecto
```

O status deve ser `"open"` quando conectado.

### 4. Webhook (automêÂ¡tico)

O Intelecto configura o webhook automaticamente ao iniciar. Ele verifica se a instêÂ¢ncia existe e se o webhook estêÂ¡ apontando para `http://intelecto:8000/webhook/whatsapp`. Se nêÂ£o estiver, configura sozinho.

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

## Verificando se estêÂ¡ tudo funcionando

### Status dos containers

```bash
docker compose ps
```

### Logs do Intelecto

```bash
docker compose logs -f intelecto
```

### Teste rêÂ¡pido

Envie uma mensagem para o nêÂºmero do WhatsApp conectado. Nos logs do Intelecto vocêÂª deve ver:

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

Neste caso, ajuste `EVOLUTION_API_URL` no `.env` para apontar para a URL acessêÂ­vel da Evolution API (ex: `http://localhost:8085`).

## Estrutura do projeto

```
ê¢âÅê¢ââ¬ê¢ââ¬ docker-compose.yml    # OrquestraêÂ§êÂ£o dos containers
ê¢âÅê¢ââ¬ê¢ââ¬ Dockerfile            # Build do app Python
ê¢âÅê¢ââ¬ê¢ââ¬ .env                  # VariêÂ¡veis de ambiente (nêÂ£o commitado)
ê¢âÅê¢ââ¬ê¢ââ¬ .env.example          # Template das variêÂ¡veis
ê¢âÅê¢ââ¬ê¢ââ¬ soul.md               # Personalidade do agente
ê¢âÅê¢ââ¬ê¢ââ¬ requirements.txt      # DependêÂªncias Python
ê¢âÅê¢ââ¬ê¢ââ¬ data/                 # Dados persistentes (lid_map.json, etc)
ê¢ââê¢ââ¬ê¢ââ¬ src/
    ê¢âÅê¢ââ¬ê¢ââ¬ main.py           # FastAPI app, webhook, lifespan
    ê¢âÅê¢ââ¬ê¢ââ¬ config.py         # ConfiguraêÂ§êÂµes (pydantic-settings)
    ê¢âÅê¢ââ¬ê¢ââ¬ agent/
    ê¢ââ   ê¢âÅê¢ââ¬ê¢ââ¬ core.py       # Orquestrador do agente
    ê¢ââ   ê¢ââê¢ââ¬ê¢ââ¬ soul.py       # Carrega soul.md
    ê¢âÅê¢ââ¬ê¢ââ¬ channels/
    ê¢ââ   ê¢ââê¢ââ¬ê¢ââ¬ whatsapp.py   # Cliente Evolution API (envio/recebimento)
    ê¢ââê¢ââ¬ê¢ââ¬ llm/
        ê¢âÅê¢ââ¬ê¢ââ¬ base.py       # Interface abstrata do LLM
        ê¢ââê¢ââ¬ê¢ââ¬ openrouter.py # ImplementaêÂ§êÂ£o OpenRouter
```

## Troubleshooting

### Mensagens nêÂ£o chegam no bot
- Verifique se a instêÂ¢ncia estêÂ¡ conectada (`connectionState` = `open`)
- Verifique os logs: `docker compose logs -f intelecto`
- O webhook êÂ© configurado automaticamente, mas confirme com:
  ```bash
  curl -H "apikey: SUA_EVOLUTION_API_KEY" http://localhost:8085/webhook/find/intelecto
  ```

### Bot recebe mas nêÂ£o responde
- Verifique os logs por erros na chamada ao OpenRouter
- Confirme que `OPENROUTER_API_KEY` estêÂ¡ vêÂ¡lida no `.env`
- Verifique se o mapeamento LID existe em `data/lid_map.json`

### InstêÂ¢ncia perdida apêÂ³s restart
- A Evolution API persiste instêÂ¢ncias nos volumes Docker (`evolution_instances`, `evolution_store`)
- Se os volumes forem removidos, êÂ© necessêÂ¡rio recriar a instêÂ¢ncia e escanear o QR Code novamente
- O webhook serêÂ¡ reconfigurado automaticamente pelo Intelecto

## Guia RêÂ¡pido

```bash
# 1. Clonar e configurar
git clone git@github.com:linnoff88-dev/intelecto-testes.git
cd intelecto-testes
cp .env.example .env
# Edite o .env com suas chaves

# 2. Subir os containers
docker compose up --build -d

# 3. Acessar o painel da Evolution API
# Abra http://localhost:8085 no navegador
# Crie a instêÂ¢ncia com o nome "intelecto"
# Escaneie o QR Code no WhatsApp (ConfiguraêÂ§êÂµes > Aparelhos conectados)

# 4. Pronto! Mande uma mensagem no WhatsApp e o bot responde.

# Comandos êÂºteis:
docker compose logs -f intelecto   # Ver logs
docker compose restart intelecto   # Reiniciar apêÂ³s trocar .env
docker compose down                # Parar tudo
```

## PrêÂ©-requisitos por sistema operacional

### Linux (Ubuntu/Debian)

```bash
# Atualizar pacotes
sudo apt update

# Instalar Docker
sudo apt install -y docker.io docker-compose-v2

# Adicionar seu usuêÂ¡rio ao grupo docker (evita usar sudo)
sudo usermod -aG docker $USER
# FaêÂ§a logout e login novamente para aplicar

# Instalar Git
sudo apt install -y git

# Verificar instalaêÂ§êÂ£o
docker --version
docker compose version
git --version
```

### Windows

1. **Docker Desktop** ê¢â¬â Baixe e instale em [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)
   - Durante a instalaêÂ§êÂ£o, habilite o **WSL 2** quando solicitado
   - ApêÂ³s instalar, abra o Docker Desktop e aguarde ele iniciar

2. **Git** ê¢â¬â Baixe e instale em [git-scm.com](https://git-scm.com/)
   - Na instalaêÂ§êÂ£o, mantenha as opêÂ§êÂµes padrêÂ£o

3. **Terminal** ê¢â¬â Use o **Git Bash** (instalado junto com o Git) ou o **PowerShell**

4. **Verificar instalaêÂ§êÂ£o** ê¢â¬â Abra o terminal e rode:
   ```bash
   docker --version
   docker compose version
   git --version
   ```

ApêÂ³s instalar os prêÂ©-requisitos, siga o [Guia RêÂ¡pido](#guia-rêÂ¡pido) acima.

## Usando Ollama (LLM local) ê¢â¬â Futuro

O Intelecto estêÂ¡ preparado para suportar o [Ollama](https://ollama.com/) como provedor de LLM local, rodando modelos diretamente na sua mêÂ¡quina sem depender de APIs externas.

### Requisitos de hardware

| Modelo | RAM mêÂ­nima | GPU (opcional) | ObservaêÂ§êÂ£o |
|--------|-----------|----------------|------------|
| 7B (ex: llama3.1:8b) | 8 GB | 4 GB VRAM | Bom para testes |
| 13B | 16 GB | 8 GB VRAM | EquilêÂ­brio qualidade/velocidade |
| 70B | 64 GB | 24+ GB VRAM | Melhor qualidade, exige hardware potente |

Com GPU NVIDIA a inferêÂªncia êÂ© significativamente mais rêÂ¡pida. Sem GPU, roda na CPU (mais lento).

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

### DiferenêÂ§as entre OpenRouter e Ollama

| | OpenRouter | Ollama |
|---|-----------|--------|
| **Onde roda** | Nuvem (API externa) | Local (sua mêÂ¡quina) |
| **Custo** | Gratuito (limitado) ou pago | Gratuito (sem limites) |
| **Privacidade** | Dados passam por terceiros | Tudo local |
| **Velocidade** | Depende da internet | Depende do hardware |
| **Modelos** | Centenas disponêÂ­veis | Precisa baixar cada um |
| **Setup** | SêÂ³ precisa de API key | Precisa de hardware adequado |

---

Projeto  - ARCKHÉe