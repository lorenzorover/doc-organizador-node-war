# ⚔️ Organizador Nodewar — Sistema Tático & Discord Sync Bot

[![Versão](https://img.shields.io/badge/VERSÃO-1.0.0-0284c7?style=flat-square)](#)
[![Status](https://img.shields.io/badge/STATUS-EM_PRODUÇÃO-16a34a?style=flat-square)](#)
[![Frontend](https://img.shields.io/badge/FRONTEND-VANILLA_JS-f59e0b?style=flat-square&logo=javascript&logoColor=black)](#)
[![Python](https://img.shields.io/badge/PYTHON-3.10+-3776ab?style=flat-square&logo=python&logoColor=white)](#)
[![Discord](https://img.shields.io/badge/DISCORD-API_GATEWAY-5865f2?style=flat-square&logo=discord&logoColor=white)](#)
[![Database](https://img.shields.io/badge/DATABASE-FIRESTORE-ffca28?style=flat-square&logo=firebase&logoColor=black)](#)
[![Deploy](https://img.shields.io/badge/DEPLOY-FIREBASE_HOSTING-0284c7?style=flat-square&logo=firebase&logoColor=white)](https://organizador-node-war.web.app/)
[![Licença](https://img.shields.io/badge/LICENÇA-MIT-64748b?style=flat-square)](#)

> Sistema reativo de gerenciamento e montagem tática de grupos (**PTs**) para **Node War** (Black Desert Online), integrado em tempo real com o Discord através de uma arquitetura serverless no **Google Firebase Firestore**.

🔗 **Acesse a aplicação online:** [https://organizador-node-war.web.app/](https://organizador-node-war.web.app/)

---

## 📌 Sumário
- [Visão Geral](#-visão-geral)
- [Como Funciona a Integração (Bot + Web App)](#-como-funciona-a-integração-bot--web-app)
- [Funcionalidades Principais](#-funcionalidades-principais)
- [Arquitetura e Fluxo de Dados](#-arquitetura-e-fluxo-de-dados)
- [Tecnologias Utilizadas](#-tecnologias-utilizadas)
- [Estrutura do Repositório](#-estrutura-do-repositório)
- [Como Configurar e Executar](#-como-configurar-e-executar)
- [Segurança & Boas Práticas](#-segurança--boas-práticas)
- [Roadmap de Desenvolvimento](#-roadmap-de-desenvolvimento)

---

## 🎯 Visão Geral

A gestão de grupos para Node War em guildas competitivas de MMO frequentemente envolve atrito operacional:
1. Membros confirmam presença ou trocam de classe em bots de Discord (ex: CoGM).
2. Líderes e shotcallers precisam sincronizar manualmente essas informações em planilhas ou quadros táteis.
3. Informações desencontradas geram atrasos na montagem e desequilíbrio de buffs de grupo.

O **Organizador Nodewar** automatiza esse fluxo conectando a interface visual de arrastar-e-soltar (**Drag & Drop**) diretamente aos eventos do Discord, refletindo alterações instantaneamente em todas as telas abertas sem necessidade de recarregar a página (`F5`).

---

## 🔄 Como Funciona a Integração: Bot do Discord + Web App

A integração opera através de um modelo **orientado a eventos (Event-Driven)** com o Firebase Firestore atuando como um hub de mensageria e banco de dados em tempo real:

```
[ Discord (Membro / CoGM) ]
          │
          ▼ (Mensagem / Slash Command)
[ Bot de Integração (Python) ]
          │ 
          ├─► 1. Sanitiza texto & normaliza fontes Unicode (NFKD)
          ├─► 2. Mapeia classe do jogo
          │
          ▼ (Grava via Firebase Admin SDK)
[ Google Firestore Cloud Database ]
          │
          ├─► Documento 'current_setup': Persistência definitiva na nuvem
          └─► Documento 'bot_queue': Fila de sincronização instantânea
          │
          ▼ (Push em tempo real via onSnapshot)
[ Aplicação Web (JavaScript / Frontend) ]
          │
          ├─► Jogador no Banco de Reservas: Classe atualizada na hora
          └─► Jogador em PT Ativa: Classe guardada em STANDBY (não quebra a PT)
```

### 1. Interceptação Inteligente no Discord
O bot em Python monitora canais designados e filtra as mensagens emitidas pelos comandos de presença e troca de classe. Ele extrai:
- O identificador/apelido do usuário.
- A classe de destino informada (com suporte especial para classes com nomes compostos como Dark Knight).

### 2. Camada de Sanitização e Normalização Unicode
Usuários no Discord frequentemente usam apelidos com colchetes, símbolos ou **fontes matemáticas estilizadas** (ex.: `[𝐃𝐚𝐤𝐫𝐚𝐫]` gerado em sites de fontes).  
O bot aplica uma decomposição algorítmica `NFKD` via biblioteca `unicodedata`, convertendo qualquer caractere matemático/estilizado em texto puro (`Dakrar`). Dessa forma, a chave de busca sempre coincide com o card cadastrado no app.

### 3. Persistência Dupla e Resiliente
- **Persistência Definitiva (`current_setup`):** O bot grava a alteração no banco mesmo que ninguém esteja com o navegador aberto.
- **Canal Realtime (`bot_queue`):** Dispara um evento imediato para notificar todos os líderes com a página aberta em milissegundos.

### 4. Inteligência Tática de Standby
Se um jogador estiver no **Banco de Reservas**, sua classe muda imediatamente.  
Se o jogador já estiver alocado em uma **PT de combate**, a alteração entra em **Standby** (aguardando confirmação do líder), evitando que a formação da guerra seja desestruturada acidentalmente no meio do planejamento.

---

## ✨ Funcionalidades Principais

- 🖱️ **Interface Drag & Drop Fluida:** Painel tático para mover jogadores entre o banco de reservas e colunas de PTs (com suporte a desktop e touch em mobile).
- ⚡ **Sincronização em Tempo Real:** Alterações feitas por um editor ou pelo bot aparecem instantaneamente para todos os visualizadores conectados.
- 🛡️ **Indicadores de Função e Buffs:** Destaque para Callers, Shais, Suportes, Linha de Frente e Backline.
- ⏱️ **Trava Anti-Stale:** Validação temporal de 45 segundos para descartar eventos residuais antigos na inicialização.
- 📸 **Exportação Rápida:** Geração de imagem do quadro tático para compartilhamento no Discord da guilda.
- 💾 **Backup JSON:** Importação e exportação manual de formações táticas completas.

---

## 🛠️ Tecnologias Utilizadas

### Frontend
- **HTML5 & CSS3:** Layout responsivo, animações sutis e variáveis de tema escuro.
- **JavaScript (ES6+ Vanilla):** Lógica reativa sem sobrecarga de frameworks externos, garantindo carregamento ultrarrápido (< 60 KB).
- **Firebase Web SDK (v8 compat):** Conexão em tempo real via listeners `onSnapshot` e autenticação.

### Bot de Integração
- **Python 3.10+:** Runtime assíncrono para manipulação de eventos.
- **discord.py:** Comunicação com o Discord Gateway via WebSockets.
- **firebase-admin:** SDK administrativa para comunicação segura e autenticada com o Firestore.
- **unicodedata & re:** Normalização de codepoints e limpeza de expressões regulares.

### Infraestrutura & Nuvem
- **Google Firebase Firestore:** Banco NoSQL distribuído e serverless.
- **Firebase Hosting:** Distribuição global de arquivos estáticos via CDN segura (HTTPS).
- **Discloud:** Plataforma de execução contínua (24/7) para o bot do Discord.

---

## 📁 Estrutura do Repositório

```bash
├── index.html                 # Ponto de entrada da interface web
├── app.js                     # Núcleo da aplicação: regras de negócio, drag & drop e listeners
├── style.css                  # Estilização visual, grid das colunas e temas dos cards
├── firestore.rules            # Regras de segurança e validação do Firestore
├── metadata.json              # Configurações do projeto
├── organizador-discord-bot/   # Módulo do Bot do Discord (Python)
│   ├── main.py                # Script principal do bot (escuta Discord + envia Firestore)
│   ├── requirements.txt       # Dependências Python (discord.py, firebase-admin)
│   └── discloud.config        # Arquivo de inicialização da hospedagem Discloud
└── README.md                  # Documentação do projeto
```

---

## 🔒 Segurança & Boas Práticas

- **Credenciais Segregadas:** Nenhuma chave privada ou segredo de API está presente no código-fonte. Todas as credenciais utilizam injeção via variáveis de ambiente (`os.getenv`).
- **Regras Granulares de Banco (`firestore.rules`):**
  - Leitura pública para que membros da guilda visualizem a formação sem necessidade de login.
  - Escrita no setup protegida por autenticação (`request.auth != null`), limitando alterações a líderes autorizados.
  - Validação estrita de tamanho de payload (`jsonData.size() < 50000`) para mitigar abusos e injeção de dados anômalos.

---

## 🗺️ Roadmap de Desenvolvimento

O projeto segue um plano de evolução para tornar o processo de montagem de Node War 100% autônomo:

```
[ Fase 1: /upload Embed ] ➔ [ Fase 2: Auto-Cadastro ] ➔ [ Fase 3: Nick Padronizado ] ➔ [ Fase 4: Auto-PTs ]
```

### 📍 Fase 1: Leitura de Cadastro Inicial (`/upload`)
- Monitorar a mensagem de confirmação emitida pelo bot de presença quando um membro novo executa o comando `/upload` pela primeira vez.
- Extrair automaticamente o apelido, nome de família e classe cadastrada do membro.

### 📍 Fase 2: Auto-Cadastro no Banco de Jogadores
- Inserir o novo membro diretamente no Banco de Reservas do app web logo após a validação do `/upload`.
- Eliminar a etapa manual onde os líderes precisavam cadastrar os membros um a um no site.

### 📍 Fase 3: Padronização Automática de Nicks no Discord
- Algoritmo para transformar o apelido comum no servidor do Discord no padrão da guilda com a fonte estilizada **Mathematical Bold Serif**:
  $$\text{Dakrar} \longrightarrow \mathbf{[𝐃𝐚𝐤𝐫𝐚𝐫]}$$
- Atualização automática do apelido do membro via permissão do bot (`member.edit(nick=...)`).

### 📍 Fase 4: Leitura de Signups de Node War & Alocação Inteligente
- Capturar o evento de edição da lista de presença da Node War (Embed de Signups do CoGM).
- Identificar a lista de inscritos confirmados (30 a 50 participantes) e suas respectivas categorias (Mainball, Shai, Frontline, Flex, etc.).
- Preencher as colunas de PTs no organizador automaticamente respeitando a distribuição tática ideal de cada função.

---

## 📄 Licença

Distribuído sob a licença MIT. Sinta-se livre para adaptar e utilizar na gestão de sua guilda.
