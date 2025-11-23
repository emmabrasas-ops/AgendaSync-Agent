Agente criado para o "Build Your First Copilot Challenge" do Azure Frontier Girls

# AgendaSync-Agent
O AgendaSync é um agente criado no Azure AI Foundry (Agent Service / Foundry Classic) que entende pedidos de lembretes em linguagem natural (ex.: “amanhã às 19h”) e cria automaticamente eventos no Google Calendar.

A automação é feita por meio de uma Azure Logic App exposta via HTTP, conectada ao Google Calendar, e consumida pelo agente através de uma OpenAPI Tool.
