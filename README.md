Com certeza! Aqui está um README para o seu projeto NexShop, formatado para o GitHub:

---

# NexShop - Backend API

![Node.js](https://img.shields.io/badge/Node.js-18+-green?style=for-the-badge&logo=nodedotjs)
![Express.js](https://img.shields.io/badge/Express.js-4.x-blue?style=for-the-badge&logo=express)
![CORS](https://img.shields.io/badge/CORS-Enabled-lightgrey?style=for-the-badge)
![Helmet](https://img.shields.io/badge/Helmet-Security-orange?style=for-the-badge)
![MIT License](https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge)

O NexShop Backend API é um serviço Node.js com Express projetado para receber dados de um SDK (frontend), calcular um score de confiança para cada evento e retornar uma recomendação (`allow`, `review` ou `deny`). Ele é ideal para sistemas que precisam de uma camada de segurança e detecção de comportamento anômalo.

## 🚀 Funcionalidades

*   **Cálculo de Score de Confiança**: Avalia o risco de cada evento com base em diversos fatores comportamentais e de dispositivo.
*   **Recomendações de Ação**: Retorna `allow` (permitir), `review` (revisar) ou `deny` (negar) com base no score de confiança.
*   **Detecção de Bots**: Lógica específica para identificar e penalizar comportamentos de bot.
*   **Armazenamento em Memória (Demo)**: Mantém um histórico recente de eventos e contagem de fingerprints para análise.
*   **Endpoints de Dashboard**: Fornece estatísticas e eventos recentes para monitoramento.
*   **Segurança**: Utiliza `helmet` para cabeçalhos de segurança e `cors` para controle de acesso.

## 📊 Lógica de Score de Risco

O score de risco é calculado com base nos seguintes critérios:

*   **Tempo de Permanência Baixo em Ações Sensíveis**: Eventos com `timeOnPageMs < 3000` em rotas como `/login`, `/checkout`, `/transfer`, `/pix`, `/reset-password` aumentam o risco em 25 pontos.
*   **Falta de Interação Humana**: Menos de 10 movimentos de mouse e nenhum clique aumentam o risco em 15 pontos.
*   **User Agent (UA) Inconsistente/Automação**:
    *   UA de Windows com idioma não-português aumenta o risco em 5 pontos.
    *   UA contendo `headless`, `phantom` ou `selenium` aumenta o risco em 25 pontos.
*   **Fuso Horário Atípico**: Fuso horário fora das regiões esperadas no Brasil aumenta o risco em 5 pontos.
*   **Repetição Anômala de Fingerprint**: Mais de 10 ocorrências do mesmo fingerprint aumentam o risco em 20 pontos.
*   **Rotas de Alto Risco**: Acesso a `/checkout`, `/pix` ou `/transfer` aumenta o risco em 10 pontos.
*   **Cliques Excessivos**: Mais de 40 cliques aumentam o risco em 10 pontos.

O score final é `100 - risco`, limitado entre 0 e 100.

### Status de Recomendação:

*   **allow**: Score >= 75
*   **review**: Score >= 50 e < 75
*   **deny**: Score < 50

## 🛠️ Tecnologias Utilizadas

*   **Node.js**
*   **Express**: Framework web para Node.js.
*   **CORS**: Middleware para habilitar Cross-Origin Resource Sharing.
*   **Helmet**: Middleware para configurar cabeçalhos HTTP de segurança.
*   **Morgan**: Logger de requisições HTTP.

## ⚙️ Instalação

1.  **Clone o repositório:**
    ```bash
    git clone https://github.com/seu-usuario/nexshop-backend.git
    cd nexshop-backend
    ```

2.  **Instale as dependências:**
    ```bash
    npm install
    ```

3.  **Execute a aplicação:**
    ```bash
    npm start
    # ou para desenvolvimento com nodemon (se configurado)
    # npm run dev
    ```
    O servidor estará rodando em `http://localhost:3000` (ou na porta definida pela variável de ambiente `PORT`).

## 📋 Endpoints da API

### `POST /api/v1/risk/score`

Recebe um payload do SDK e retorna o score de confiança e a recomendação.

*   **Corpo da Requisição (JSON):**
    ```json
    {
      "client": {
        "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.127 Safari/537.36",
        "language": "pt-BR",
        "timezone": "America/Sao_Paulo",
        "ip": "192.168.1.1"
      },
      "behavior": {
        "timeOnPageMs": 5000,
        "mouseMoves": 150,
        "clicks": 5,
        "mouse": "human"
      },
      "fingerprint": "a1b2c3d4e5f6",
      "page": {
        "path": "/checkout/step1"
      }
    }
    ```
    *Nota: Se `behavior.mouse` for "bot", `fingerprint` será forçado para `true` e `timeOnPageMs` será limitado a 2000ms para simular comportamento de bot.*

*   **Resposta (JSON):**
    ```json
    {
      "ok": true,
      "score": 90,
      "status": "allow",
      "reasons": []
    }
    ```

### `GET /api/v1/dashboard/stats`

Retorna estatísticas agregadas dos últimos eventos.

*   **Resposta (JSON):**
    ```json
    {
      "ok": true,
      "totalEvents": 123,
      "window": 200,
      "averageScore": 85.5,
      "distribution": {
        "allow": 180,
        "review": 15,
        "deny": 5
      },
      "topReasons": [
        { "reason": "sem_interacao_humana", "count": 10 },
        { "reason": "rota_alto_risco", "count": 8 }
      ]
    }
    ```

### `GET /api/v1/events/recent`

Retorna os 50 eventos mais recentes.

*   **Resposta (JSON):**
    ```json
    {
      "ok": true,
      "events": [
        {
          "id": "1678886400000-abcde",
          "ts": "2023-03-15T10:00:00.000Z",
          "score": 95,
          "status": "allow",
          "reasons": [],
          "payload": { /* ... */ },
          "ip": "192.168.1.100",
          "ua": "Mozilla/5.0 ..."
        }
        // ...
      ]
    }
    ```

### `GET /api/v1/health`

Endpoint de healthcheck para verificar o status da API.

*   **Resposta (JSON):**
    ```json
    {
      "ok": true,
      "uptime": 12345.67,
      "now": "2023-03-15T10:00:00.000Z"
    }
    ```

### `GET /api/v1/clients`

Retorna uma lista de nomes de clientes (apenas para demonstração).

*   **Resposta (JSON):**
    ```json
    {
      "ok": true,
      "clients": [
        "Ana Silva",
        "Bruno Oliveira",
        "Carla Souza",
        // ...
      ]
    }
    ```

## 🤝 Contribuição

Contribuições são bem-vindas! Sinta-se à vontade para abrir issues ou enviar pull requests.

## 📄 Licença

Este projeto está licenciado sob a Licença MIT. Consulte o arquivo [LICENSE](LICENSE) para mais detalhes.

---
