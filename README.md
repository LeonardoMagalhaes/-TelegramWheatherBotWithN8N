# 🤖 Workflow Chatbot Telegram — Previsão do Tempo

Workflow N8N que integra um bot do Telegram com a API do OpenWeatherMap para consultar a temperatura atual de cidades brasileiras.

## 📋 Visão Geral

O usuário envia o nome de uma cidade pelo Telegram e recebe de volta a temperatura atual e a descrição do clima, em português.

## 🔧 Pré-requisitos

| Recurso | Descrição |
|---|---|
| **N8N** | Instância do N8N rodando (self-hosted ou cloud) |
| **Telegram Bot Token** | Criado via [@BotFather](https://t.me/BotFather) e configurado como credencial no N8N |
| **OpenWeatherMap API Key** | Obtida em [openweathermap.org](https://openweathermap.org/api) (plano gratuito é suficiente) |

## 🗺️ Fluxo do Workflow

```
Telegram Trigger
      │
      ▼
Formatar Entrada ─────► Validar Resposta
                              │
                     ┌────────┴────────┐
                     ▼                  ▼
            OpenWeather API        Enviar Erro
                     │            (cidade inválida)
                     ▼
             Code Fallback
                     │
                     ▼
        Preparar Mensagem Final
                     │
                     ▼
          Enviar Temperatura
           (resposta ao usuário)
```

## 🔍 Descrição dos Nodes

### 1. Telegram Trigger

- **Tipo:** `telegramTrigger`
- **Função:** Escuta mensagens recebidas pelo bot do Telegram.
- **Evento monitorado:** `message`

### 2. Formatar Entrada

- **Tipo:** `set`
- **Função:** Extrai e formata os dados da mensagem do usuário.
- **Campos gerados:**
  - `queue` — nome da cidade formatado em minúsculo com sufixo `,BR` (ex.: `são paulo,BR`)
  - `chatId` — ID do chat do Telegram para enviar a resposta

### 3. Validar Resposta

- **Tipo:** `if`
- **Função:** Verifica se a resposta da API retornou `cod == 200` (sucesso).
- **Saídas:**
  - ✅ **True** → Segue para `OpenWeather API`
  - ❌ **False** → Segue para `Enviar Erro`

### 4. OpenWeather API

- **Tipo:** `httpRequest`
- **Função:** Consulta a API do OpenWeatherMap.
- **Endpoint:** `https://api.openweathermap.org/data/2.5/weather`
- **Parâmetros:**
  - `queue` — cidade formatada (ex.: `são paulo,BR`)
  - `units` — `metric` (Celsius)
  - `lang` — `pt_br` (respostas em português)
  - `appid` — chave da API (configurar via credencial ou variável de ambiente)

### 5. Code Fallback

- **Tipo:** `code` (JavaScript)
- **Função:** Monta uma mensagem determinística de fallback com a temperatura.
- **Exemplo de saída:**
  ```
  🌤️ A temperatura em São Paulo é de 25°C.
  ```
- **Campos gerados:** `chatId`, `cidade`, `temp`, `descricao`, `mensagemFallback`

### 6. Preparar Mensagem Final

- **Tipo:** `set`
- **Função:** Define a mensagem final a ser enviada.
- **Lógica:** Usa o texto gerado (se disponível) ou cai para o `mensagemFallback`.
  ```
  mensagemFinal = $json.text || mensagemFallback
  ```

### 7. Enviar Temperatura

- **Tipo:** `telegram`
- **Função:** Envia a mensagem final com a previsão do tempo ao usuário no Telegram.

### 8. Enviar Erro

- **Tipo:** `telegram`
- **Função:** Envia uma mensagem de erro caso a cidade não seja encontrada.
- **Mensagem:**
  ```
  ❌ Cidade não encontrada. Use o formato: Cidade, UF (ex.: São Paulo, SP).
  ```

## 💬 Como Usar

1. Abra o Telegram e inicie uma conversa com o bot.
2. Envie o nome de uma cidade brasileira:
   ```
   São Paulo, SP
   ```
3. O bot responderá com a temperatura atual:
   ```
   🌤️ A temperatura em São Paulo é de 25°C.
   ```

## ⚙️ Configuração

1. **Importe o workflow** no N8N usando o arquivo JSON.
2. **Configure as credenciais do Telegram** com o token do bot.
3. **Insira a API Key do OpenWeatherMap** no parâmetro `appid` do node `OpenWeather API`.
4. **Ative o workflow** no N8N.

## 📝 Observações

- O workflow adiciona automaticamente `,BR` ao nome da cidade, limitando a busca ao Brasil.
- A unidade de temperatura é Celsius (`metric`).
- As respostas da API vêm em português (`pt_br`).
- O workflow possui um mecanismo de fallback para garantir que o usuário sempre receba uma resposta, mesmo sem enriquecimento por IA.
