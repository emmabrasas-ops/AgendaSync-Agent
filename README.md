# AgendaSync — Agente de Lembretes com Google Calendar (Azure AI Foundry + Logic Apps)

ID do Agente: asst_zAf2cR8K1jWvzXMeoyuewCV5

## 📌 Descrição do projeto

O **AgendaSync** é um agente criado no **Azure AI Foundry (Agent Service / Foundry Classic)** que entende pedidos de lembretes em linguagem natural (ex.: “amanhã às 19h”) e cria automaticamente eventos no **Google Calendar**.

A automação é feita por meio de uma **Azure Logic App** exposta via HTTP, conectada ao Google Calendar, e consumida pelo agente através de uma **OpenAPI Tool**.

---

## 🎯 Objetivo do agente

1. Receber pedidos de lembretes em português de forma natural.  
2. Interpretar corretamente título, data e hora (incluindo termos relativos: hoje/amanhã).  
3. Confirmar a data interpretada quando necessário.  
4. Enviar os dados para a Logic App via OpenAPI Tool.  
5. Criar o evento no Google Calendar de forma automática.  
6. Confirmar ao usuário que o evento foi criado com sucesso.  

---

## 🧩 Arquitetura / Fluxo geral

**Usuário → Agente (Foundry) → OpenAPI Tool → Logic App (HTTP Trigger) → Google Calendar**

1. O usuário conversa com o **AgendaSync** no Playground.  
2. O agente extrai:
   - `titulo`
   - `data_hora_inicio`
   - `data_hora_fim`
   - `descricao` (opcional)
3. O agente chama a **CalendarTool** (OpenAPI Tool).  
4. A Tool envia um POST para a **Logic App (HTTP Request Trigger)**.  
5. A Logic App usa o conector do **Google Calendar** para criar o evento.  
6. O agente confirma o sucesso.  

---

## ✅ Pré-requisitos

- Conta Azure com acesso ao **Azure AI Foundry / Azure AI Studio Classic**
- Um projeto criado no Foundry
- Um deployment de modelo (ex.: GPT-4.1-mini, GPT-4o ou equivalente)
- Acesso a **Azure Logic Apps**
- Conta Google com Google Calendar habilitado

---

## 🚀 Passo a passo para recriar o projeto

### Parte 1 — Criar a Logic App (backend do evento)

1. Acesse **Azure Portal → Logic Apps**.
2. Clique em **Create Logic App**.
3. Crie uma Logic App (Consumption ou Standard).
4. Abra o **Designer** e crie um workflow em branco.
5. Escolha o gatilho:
   - **Request → “When an HTTP request is received”**
   - Esse gatilho cria uma URL pública para disparo por HTTP.
6. No gatilho, adicione um schema JSON:
<img width="609" height="415" alt="image" src="https://github.com/user-attachments/assets/c56d170a-9229-4057-8dee-69c280ed560c" />

7.Adicione a ação:

Google Calendar → Create an event

8. Configure os campos:

Calendar ID: seu calendário

Summary: titulo

Description: descricao

Start time: data_hora_inicio

End time: data_hora_fim

9. Adicione a ação final:

Response (200 OK) retornando JSON de sucesso.

<img width="626" height="516" alt="image" src="https://github.com/user-attachments/assets/61b89099-c091-483e-b04b-372f04017349" />

10. Salve o workflow e copie a URL do gatilho HTTP (vai ser algo como):

<img width="1642" height="778" alt="image" src="https://github.com/user-attachments/assets/f8d2d5b7-5f91-4b6d-a12e-db925e4fc762" />

---

## Aqui foi necessário um passo extra, pois deu um erro referente a Timezone. 
Adicionei então 2 Compose. 

## ⏱️ Ajuste de timezone na Logic App (Compose e Compose 1)

Para garantir que o Google Calendar receba **sempre o horário do Brasil**, adicionamos dois passos **Compose** antes da ação “Criar um evento”.

### Por que isso é necessário?

- O agente envia `data_hora_inicio` e `data_hora_fim` como texto.
- A Logic App pode interpretar esse texto como **UTC**.
- O Google Calendar cria o evento com base no fuso que recebeu.
- Resultado: o evento aparece em horário/dia diferente do esperado.

👉 O Compose serve para **converter o horário recebido para o fuso correto do Brasil** antes de criar o evento.

---

### Como configurar

No fluxo da Logic App, o fluxo fica assim:

**When an HTTP request is received**  
→ **Compose (start_local)**  
→ **Compose 1 (end_local)**  
→ **Criar um evento**  
→ **Response**

---

### 1) Compose (start_local)

<img width="612" height="288" alt="image" src="https://github.com/user-attachments/assets/48af5ae9-03f9-4574-8f16-763a37c31097" />


### 2) Compose 1 (end_local)

<img width="625" height="219" alt="image" src="https://github.com/user-attachments/assets/ac2c3332-4ac8-485f-b8e9-a1430a5e072d" />


O teste então retornou bem-sucedido.

<img width="1917" height="862" alt="image" src="https://github.com/user-attachments/assets/6f9b9c77-eb32-4360-8c89-faf522a278e8" />

Informações do "AgendaSync" no LogicApp:

<img width="415" height="297" alt="image" src="https://github.com/user-attachments/assets/431a983c-9543-437c-92d6-4d470aa9b505" />


## 🚀 Continuação do Passo a passo para recriar o projeto

### Parte 2 — Criar o Projeto no Azure

IMPORTANTE: Precisei desabilitar o NewFoundry pois estava encontrando sucessivos erros na criação do agente e integração com a LogicApp. Eu não consegui adicionar a Tool com a info do LogiApp e associá-la ao agente no NewFoundry. Desabilitei e liguei o Classi, e funcionou. Steps a seguir:

1) Crie o Projeto (AgendaSync)
   <img width="1148" height="690" alt="image" src="https://github.com/user-attachments/assets/c8677f4b-ba1d-40e3-838b-57d441e117e6" />

Endpoint do Projeto: <https://agendasync-resource.services.ai.azure.com/api/projects/agendasync>

2) Crie o Agente dentro do Projeto

   <img width="1686" height="469" alt="image" src="https://github.com/user-attachments/assets/553428b0-c97c-4dce-abc4-480108fc64a3" />

3) Crie as Instruções para o agente:

"REGRAS OBRIGATÓRIAS DE DATA (NÃO QUEBRE)

"- Considere que HOJE é 23/11/2025 (America/Sao_Paulo).
- “amanhã” = 24/11/2025.
- Se o usuário usar termos relativos (hoje, amanhã, semana que vem, próximo mês etc.):
    1) Primeiro diga a data completa que você entendeu (com dia/mês/ano).
    2) Pergunte “Posso criar assim?”.
    3) Só chame CalendarTool depois do usuário confirmar.
- Se o usuário não disser o ano e não usar termo relativo, use o ano atual (2025).
- Nunca envie datas em 2024 ou anos passados.
- Sempre mande com fuso explícito: yyyy-MM-ddTHH:mm:ss-03:00."

4) Vá em "Ações" e selecione "Adicionar ação" e selecione "Ferramenta especificada pelo OpenAPI 3.0"

 <img width="1108" height="678" alt="image" src="https://github.com/user-attachments/assets/22a71ca6-a9b3-4bf3-a828-f37e1ee7baf1" />

5) Adicione Nome e Descrição da Ferramenta. (CalendarTool - Google Agenda)

<img width="1245" height="846" alt="image" src="https://github.com/user-attachments/assets/648d8e3e-274a-4b65-a707-64ed58e2cad8" />

6) Selecione "Anônimo" em Método de Autenticação e insira o código JSON para a Tool.

<img width="1267" height="862" alt="image" src="https://github.com/user-attachments/assets/4117ab68-dff9-4110-a3ac-7a594f3bc79b" />

### OpenAPI schema da CalendarTool (Foundry Classic)

Na criação da tool no **Azure AI Foundry Classic** (Tools → Create custom tool → OpenAPI tool), cole o schema abaixo.

> ⚠️ **Importante:** se este repositório for público, **não publique a chave `sig` real**.  
> Substitua `SEU_SIG_AQUI` pelo valor do parâmetro `sig` da URL da sua Logic App apenas no Foundry.

```json
{
  "openapi": "3.0.1",
  "info": {
    "title": "CalendarTool",
    "version": "1.0",
    "description": "Cria eventos no Google Calendar via Logic Apps"
  },
  "servers": [
    {
      "url": "https://prod-16.brazilsouth.logic.azure.com:443"
    }
  ],
  "paths": {
    "/workflows/a6f5ae9fba664cdba33a77bff7a7fd05/triggers/When_an_HTTP_request_is_received/paths/invoke": {
      "post": {
        "operationId": "criar_evento_google_calendar",
        "summary": "Criar evento no Google Calendar",
        "parameters": [
          {
            "name": "api-version",
            "in": "query",
            "required": true,
            "schema": { "type": "string", "default": "2016-10-01" }
          },
          {
            "name": "sp",
            "in": "query",
            "required": true,
            "schema": { "type": "string", "default": "/triggers/When_an_HTTP_request_is_received/run" }
          },
          {
            "name": "sv",
            "in": "query",
            "required": true,
            "schema": { "type": "string", "default": "1.0" }
          },
          {
            "name": "sig",
            "in": "query",
            "required": true,
            "schema": {
              "type": "string",
              "default": "SEU_SIG_AQUI"
            }
          }
        ],
        "requestBody": {
          "required": true,
          "content": {
            "application/json": {
              "schema": {
                "type": "object",
                "required": ["titulo", "data_hora_inicio", "data_hora_fim"],
                "properties": {
                  "titulo": { "type": "string" },
                  "data_hora_inicio": {
                    "type": "string",
                    "description": "yyyy-MM-ddTHH:mm:ss (America/Sao_Paulo)"
                  },
                  "data_hora_fim": {
                    "type": "string",
                    "description": "yyyy-MM-ddTHH:mm:ss (America/Sao_Paulo)"
                  },
                  "descricao": { "type": "string" }
                }
              }
            }
          }
        },
        "responses": {
          "200": { "description": "OK" }
        }
      }
    }
  }
}
```
Ferramenta adicionada ao Agente!

<img width="417" height="126" alt="image" src="https://github.com/user-attachments/assets/891e7164-0e48-40fb-adf7-c21c7a3b0293" />

## 🚀 Continuação do Passo a passo para recriar o projeto

### Parte 3 — Testar o Agente

1) Dar um prompt de teste conforme o exemplo:

## 🤖 Como o AgendaSync funciona

O **AgendaSync** é um agente conversacional no Azure AI Foundry Classic que cria lembretes no Google Calendar a partir de mensagens em linguagem natural.

### Fluxo de funcionamento

1. **O usuário pede um lembrete em português**  
   Exemplo:  
   “Crie um lembrete para estudar inglês amanhã às 19h.”

2. **O agente interpreta o pedido**  
   Ele extrai automaticamente:
   - **titulo**: o que será feito (ex.: “Estudar inglês”)
   - **data_hora_inicio**: data e hora de início
   - **data_hora_fim**: data e hora de fim (se o usuário não disser duração, o agente assume 1 hora)
   - **descricao** (opcional)

3. **Tratamento de datas relativas**  
   Se o usuário usar termos como “hoje”, “amanhã”, “semana que vem”, o agente:
   - converte para uma data completa (dia/mês/ano)
   - usa o **ano atual**
   - **confirma com o usuário antes de criar**
   
   Exemplo de confirmação:  
   “Entendi: Estudar inglês em 24/11/2025 às 19:00 até 20:00. Posso criar assim?”

4. **Chamada da Tool (CalendarTool)**  
   Após a confirmação, o agente chama a ação **CalendarTool**, enviando um POST para a Logic App com o body:

   ```json
   {
     "titulo": "Estudar inglês",
     "data_hora_inicio": "2025-11-24T19:00:00-03:00",
     "data_hora_fim": "2025-11-24T20:00:00-03:00",
     "descricao": ""
   }
´´´

A Logic App cria o evento no Google Calendar
A Logic App recebe a solicitação via HTTP, ajusta o timezone com os passos Compose e Compose 1, e executa a ação Google Calendar → Create an event.

O agente confirma o sucesso ao usuário
Ao receber a resposta “OK” da Logic App, o agente responde confirmando que o evento foi criado e adicionado ao Google Calendar.

Resultado final

✅ O usuário cria lembretes de forma natural (sem preencher formulário).

✅ O agente transforma texto em evento pronto.

✅ O evento aparece automaticamente na agenda Google.

##### Exemplo real de teste com prints:

<img width="1132" height="508" alt="image" src="https://github.com/user-attachments/assets/3221f24c-c356-4fcd-8f0a-4507cea8b380" />

Evento criado com sucesso na agenda:

<img width="1821" height="569" alt="image" src="https://github.com/user-attachments/assets/71648e40-10a0-4c0f-861e-e1569dfccbec" />




