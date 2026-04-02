# 🚀 Máquina de Vendas Automática: Ecossistema de Afiliados Mercado Livre (End-to-End)

![Status](https://img.shields.io/badge/Status-Produção-success?style=for-the-badge)
![FinOps](https://img.shields.io/badge/FinOps-Zero_Cost-brightgreen?style=for-the-badge)
![n8n](https://img.shields.io/badge/n8n-Docker_Self--Hosted-orange?style=for-the-badge&logo=n8n)
![Google Apps Script](https://img.shields.io/badge/Google_Apps_Script-Serverless-blue?style=for-the-badge&logo=google)
![Redis](https://img.shields.io/badge/Redis-In--Memory_DB-red?style=for-the-badge&logo=redis)
![AI](https://img.shields.io/badge/AI-Llama_3.3_(Groq)-purple?style=for-the-badge)

## 📌 Visão Geral do Projeto

Este projeto consiste em uma arquitetura de automação de marketing ponta a ponta (End-to-End), projetada para otimizar, escalar e automatizar 100% o processo de divulgação de produtos do programa de afiliados do Mercado Livre.

O sistema resolve problemas complexos de automação web — como **expiração de sessões (cookies)**, **extração de dados dinâmicos (scraping)** e **geração de copy em tempo real** — integrando um Painel Web Serverless, um banco de dados em memória (Redis) e um Motor de Automação (n8n) com **ingestão Omnichannel** (Painel Web e Telegram).

---

## 💡 FinOps e Arquitetura Zero-Cost (Open-Source Stack)

Um dos pilares fundamentais deste projeto foi o design focado em **máxima eficiência com custo zero (Zero-Cost Architecture)**. Todas as ferramentas, bancos de dados e modelos de Inteligência Artificial foram estrategicamente selecionados para entregar performance a nível *Enterprise* sem gerar custos mensais com licenças de SaaS ou APIs pagas:

*   **Google Apps Script & Sheets:** Substituem servidores web pagos e bancos de dados em nuvem, rodando de forma 100% gratuita na infraestrutura do Google.
*   **n8n, Evolution API & Redis:** Ferramentas *Open-Source* / *Fair-Code* auto-hospedadas (Self-hosted) via Docker, eliminando mensalidades de plataformas como Make/Zapier ou APIs oficiais de WhatsApp.
*   **Inteligência Artificial (Groq):** Utilização do *Tier* gratuito da Groq API para rodar o modelo Llama 3.3, evitando custos por token da OpenAI (ChatGPT).
*   **Telegram Bot API & Ngrok:** Túneis reversos e mensageria de controle utilizando camadas gratuitas vitalícias.

---

## 🏗️ Arquitetura do Sistema e Ingestão Omnichannel

A arquitetura foi desenhada com múltiplas portas de entrada (Triggers), permitindo tanto o processamento em lote (Batch) quanto o processamento em tempo real sob demanda (Event-Driven).

### 1. Ingestão em Lote: Painel de Controle e Gestão de Estado (Serverless)
O cérebro do agendamento não roda no navegador, mas sim nos servidores do Google.
*   **Fila Persistente (Google Sheets):** Atua como um banco de dados NoSQL de fila contínua.
*   **Triggers Dinâmicos:** Ao clicar em "Ativar Robô" no Front-end, o sistema faz uma chamada à API do Google Apps Script para criar **Acionadores Baseados em Tempo** diretamente no Google Cloud. 
*   **Independência de Hardware:** O servidor do Google acorda automaticamente nos turnos configurados (ex: 10h, 14h e 19h), lê a planilha usando um **Ponteiro de Memória** (última linha processada) e dispara o Webhook para o Back-end. O usuário pode fechar a aba e desligar o PC sem interromper o fluxo.

### 2. Ingestão em Tempo Real: Telegram Bot (Event-Driven)
Para links de oportunidade ("Ofertas Relâmpago"), o sistema conta com uma entrada via **Telegram Bot API**.
*   **Processamento "On-the-go":** O administrador pode simplesmente colar o link do Mercado Livre no chat do bot pelo celular.
*   **Normalização de Payload (Polimorfismo):** O workflow do n8n possui inteligência de roteamento. Seja o dado vindo do Webhook do Google (lote) ou do Bot do Telegram (tempo real), um nó de transformação normaliza o JSON de entrada. Ele padroniza variáveis estruturais (`url`, `chat_id`, `username`) para que ambas as fontes utilizem exatamente a mesma esteira de processamento (Scraping, IA e Distribuição), garantindo o princípio *DRY (Don't Repeat Yourself)*.

### 3. Motor de Automação e Hospedagem (n8n + Docker)
*   **Infraestrutura:** O core do processamento roda no **n8n**, hospedado em um ambiente **Docker** (Self-hosted via VPS local/nuvem). Isso garante isolamento e resiliência contra quedas.
*   **Exposição Segura:** O webhook de entrada é exposto à internet através de um túnel reverso seguro (Ngrok/Cloudflare Tunnels).

---

## 🍪 Destaque Técnico de Engenharia: Sistema "Self-Healing" de Cookies

A API de afiliados do Mercado Livre expira sessões e rotaciona tokens de segurança constantemente. Para contornar isso e criar links via API de forma autônoma, desenvolvi um **Loop de Renovação Automática de Cookies com Redis**.

**Como o algoritmo funciona passo a passo:**
1.  **Recuperação de Sessão (Redis):** O n8n consulta o banco **Redis** em memória para resgatar a última *string* de cookies válida (`cookies_mercadolivre`).
2.  **Handshake com o Servidor:** O n8n faz a requisição injetando esses cookies resgatados no Header.
3.  **Interceptação e Merge (O Pulo do Gato):** O servidor responde com novos cabeçalhos `set-cookie`. Um nó **JavaScript Customizado** atua como um "mesclador" (Merger) em memória:
    *   Faz o *parsing* do dicionário antigo e do novo.
    *   Sobrescreve apenas os tokens expirados e mantém chaves de sessão antigas vitais intactas.
4.  **Persistência e Bypass:** A nova string reconstruída é salva de volta no Redis e utilizada na requisição final de `createLink`, garantindo 100% de sucesso e evasão de bloqueios de segurança.

---

## 🕷️ Web Scraping Estruturado e Extração de Dados

Para evitar chamadas complexas a APIs de dados pagas, o sistema atua como um *Scraper Dinâmico*. O n8n faz o download do HTML bruto da página original do produto e, utilizando Regex avançado, extrai:
*   **OG Tags:** Captura a imagem principal (`og:image`) e o título original (`og:title`).
*   **Parsing de JSON Embutido:** Localiza o JSON injetado no código-fonte da página para extrair o **Preço Original** e o **Preço Promocional** de forma exata, calculando a ancoragem de desconto (De: X, Por: Y).
*   **Tratamento de Exceções:** Se o produto estiver sem preço (esgotado ou link inválido), o fluxo desvia para uma rota de erro e notifica o emissor (Painel ou Telegram) imediatamente.

---

## 🧠 Inteligência Artificial (Copywriting de Alta Conversão)

O sistema integra LLMs de ponta para gerar *Hooks* (chamadas) persuasivos dinamicamente:
*   **Provedor e Modelo:** Utiliza a API da **Groq** rodando o modelo open-source **Llama-3.3-70b-versatile**.
*   **Engenharia de Prompt:** O LLM recebe o título higienizado do produto e gera uma chamada curta, em CAPS LOCK, de até 7 palavras (ex: "🔥 OFERTA RELÂMPAGO IMPERDÍVEL!"), formatando a postagem para máximo CTR (Click-Through Rate).

---

## 📲 Distribuição e Auditoria Multicanal

Com o payload completo enriquecido (Imagem + Link + IA + Preços), o n8n orquestra a saída:
1.  **Evolution API (WhatsApp):** O sistema se conecta a uma instância da Evolution API (Baileys), enviando a mídia e a legenda formatada diretamente para grupos VIPs de ofertas no WhatsApp.
2.  **Telegram Bot API (Log e Auditoria):** Simultaneamente, o administrador recebe o feedback no chat do Telegram confirmando a postagem, servindo como log de sucesso em tempo real com o link gerado em anexo.

---

## 🖼️ Arquitetura Visual do Projeto

> **Aviso de Segurança:** Todas as credenciais, IDs de chats, URLs de Webhooks e tokens de sessão presentes neste repositório foram higienizados e ofuscados para fins de demonstração pública.

### 1. Painel Web de Controle (Front-end)
<img width="1907" height="924" alt="front-end" src="https://github.com/user-attachments/assets/57afcbdc-b86d-46ac-a821-f4af4b2417b8" />
> Painel desenvolvido em HTML/CSS nativo integrado ao Google Apps Script. Exibe cronômetros em tempo real sincronizados com os Triggers do servidor e permite gerenciamento total de lotes e turnos.

### 2. Orquestração de Dados (Workflow n8n)
<img width="1803" height="438" alt="workflow" src="https://github.com/user-attachments/assets/7f6ec436-b671-48d5-8c1f-9473436ba0c5" />
> Visão completa do pipeline. Destaque para as rotas condicionais (If Nodes), gerenciamento de estado no Redis (Início do fluxo) e requisições HTTP seguidas da geração de conteúdo com IA (Groq).

### 3. Resultado Final em Produção
<img width="1296" height="859" alt="tela whats" src="https://github.com/user-attachments/assets/924cc8ae-7d70-47ce-ba24-b70e95f1965e" />
> Demonstração da mensagem final gerada de forma 100% autônoma, entregue no WhatsApp com formatação rica (negrito, tachado), ancoragem de preço, hook da IA e imagem nativa do produto.
