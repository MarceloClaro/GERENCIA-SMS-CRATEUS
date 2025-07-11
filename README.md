
# *MedSMS-Crateús AI - Painel Médico (Documentação Técnica)*
---

Este documento fornece uma análise técnica detalhada do projeto MedSMS-Crateús AI, um painel de gerenciamento médico completo. O objetivo é servir como um guia para desenvolvedores, arquitetos de software e equipes de avaliação técnica, facilitando a compreensão, a manutenção e a reprodução do ambiente de desenvolvimento.

---

## 1. Visão Geral do Projeto

O **MedSMS-Crateús AI** é uma Single-Page Application (SPA) projetada para otimizar a gestão de clínicas e consultórios médicos. A plataforma centraliza o gerenciamento de pacientes, agendamentos, procedimentos e finanças, integrando-se nativamente com a **API Google Gemini** para fornecer insights avançados, automação e suporte à decisão clínica através de uma interface rica e reativa.

A aplicação foi construída para funcionar de forma robusta no lado do cliente, utilizando o **IndexedDB** para persistência de dados, o que garante funcionalidade offline e uma experiência de usuário rápida e fluida.

---

## 2. Principais Funcionalidades

A plataforma é modular e abrange diversas áreas da gestão clínica:

-   **Dashboard Principal**: Visão centralizada com estatísticas-chave, calendário de agendamentos, e uma "Central de Operações Diárias" para gerenciar a fila de espera e ocorrências do dia.
-   **Análise Estratégica com IA**: O Dashboard apresenta múltiplos gráficos de análise (financeiro, desempenho de médicos, etc.), cada um com uma **Análise SWOT** (Forças, Fraquezas, Oportunidades, Ameaças) gerada dinamicamente pela IA para fornecer insights de gestão.
-   **Gestão de Cadastros (CRUD Completo)**: Módulos completos para gerenciar todas as entidades do sistema:
    -   Pacientes
    -   Médicos
    -   Procedimentos e Tipos de Procedimentos
    -   Locais de Atendimento e Municípios
    -   Ocorrências de Agendamento (ex: "Paciente chegou", "Em atendimento")
    -   Campanhas de Saúde
    -   Tabelas de Preços (ex: SUS, Particular)
-   **Agenda Diária Interativa**: Uma visualização de agenda por dia que permite a criação e edição de agendamentos inline, diretamente nos horários vagos.
-   **Assistente Clínico (IA)**: Uma interface de chat que utiliza o contexto completo do banco de dados para responder a perguntas em linguagem natural, como "Qual o histórico do paciente X?" ou "Mostre a agenda do Dr. Y para amanhã".
-   **Central de Automação (IA)**: Módulo proativo que usa a IA para gerar e sugerir mensagens personalizadas para pacientes, incluindo lembretes de consulta, instruções de preparo para exames, acompanhamento pós-consulta e convites para campanhas.
-   **Relatórios Gerenciais Customizáveis**: Ferramenta de BI que permite filtrar agendamentos por múltiplos critérios (período, médico, local, etc.), visualizar os dados em uma tabela com colunas selecionáveis e exportar o resultado para **XLSX**.
-   **Gerenciamento do Banco de Dados**: Ferramenta para **exportar** um backup completo, **importar** um backup (substituindo todos os dados atuais), e **resetar** o banco de dados para o estado inicial de demonstração.

---

## 3. Tech Stack & Ferramentas

-   **Frontend Framework**: **React 19** (com Hooks) e **TypeScript**.
-   **API de Inteligência Artificial**: **Google Gemini API** (via SDK `@google/genai`).
-   **Estilização**: **Tailwind CSS**, carregado via CDN para uma abordagem utility-first sem a necessidade de um processo de build para o CSS.
-   **Banco de Dados (Client-Side)**: **IndexedDB**, proporcionando uma base de dados NoSQL robusta no navegador.
-   **Manipulação de Planilhas**: **`xlsx`** (SheetJS) para importação e exportação de dados nos formatos CSV e XLSX.
-   **Ambiente de Desenvolvimento**: **Vite.js**, configurado para servir os arquivos estáticos e gerenciar variáveis de ambiente. A aplicação utiliza um **`importmap`** no `index.html`, eliminando a necessidade de um passo de *bundling* local durante o desenvolvimento.
-   **Gerenciador de Pacotes**: `npm`.

---

## 4. Arquitetura e Padrões de Design

A arquitetura do projeto foi pensada para ser modular, escalável e de fácil manutenção, mesmo sendo uma aplicação client-side.

-   **Gerenciamento de Estado Global (React Context API)**:
    -   O coração da aplicação é o `AppContext.tsx`. Ele funciona como um *store* centralizado que provê um estado global para toda a árvore de componentes.
    -   Ele é responsável por:
        1.  Manter o estado de todas as entidades de dados (pacientes, agendamentos, etc.).
        2.  Fornecer todas as funções de manipulação de dados (CRUD) que interagem com o `utils/db.ts`.
        3.  Controlar o estado e a visibilidade de todos os modais da aplicação.
        4.  Gerenciar a view (tela) atualmente exibida.
    -   Qualquer componente pode acessar o estado e as funções através do hook customizado `useAppContext()`.

-   **Persistência de Dados (IndexedDB Abstraction)**:
    -   Toda a lógica de interação com o IndexedDB é abstraída no arquivo `utils/db.ts`.
    -   Este módulo é responsável por inicializar o banco, criar e versionar os *object stores* (tabelas), e fornecer uma API simples para as operações CRUD (`getAllData`, `saveData`, `deleteData`, `bulkPut`).
    -   Na primeira execução, o `db.ts` também popula (seeds) o banco de dados com dados de exemplo do `data/mock.ts`.

-   **Service Layer para IA**:
    -   Toda a comunicação com a API Gemini é encapsulada em `services/geminiService.ts`.
    -   Este serviço exporta funções especializadas para cada funcionalidade de IA, cada uma com *system instructions* (prompts de sistema) cuidadosamente elaborados para garantir respostas precisas e no formato desejado (texto ou JSON).

-   **Fluxo de Dados Unidirecional**:
    1.  **Carregamento**: `AppProvider` -> `AppContext` -> `db.ts` -> IndexedDB.
    2.  **Renderização**: O estado do `AppContext` é passado como props para os componentes.
    3.  **Ações do Usuário**: Componente de UI (ex: `PatientForm`) -> Chama função do `AppContext` (ex: `savePatient`) -> `AppContext` atualiza o `db.ts` -> `AppContext` atualiza seu próprio estado com `setState` -> React re-renderiza os componentes afetados.

---

## 5. Estrutura de Diretórios

A estrutura de arquivos foi organizada por funcionalidade para manter a coesão e facilitar a localização do código.

```
/
├── public/
├── src/ (ou raiz do projeto)
│   ├── components/
│   │   ├── *List.tsx          # Componentes de listagem (ex: PatientList)
│   │   ├── *Form.tsx          # Componentes de formulário em modais (ex: PatientForm)
│   │   ├── AiAssistant.tsx      # UI do chat com a IA
│   │   ├── AutomationCenter.tsx # UI das sugestões de automação
│   │   ├── Dashboard.tsx        # Painel principal com gráficos e análises
│   │   ├── ReportsView.tsx      # Ferramenta de relatórios
│   │   ├── Modal.tsx            # Componente genérico reutilizável para modais
│   │   ├── Sidebar.tsx          # Barra de navegação lateral
│   │   └── ...                  # Outros componentes de UI
│   │
│   ├── context/
│   │   └── AppContext.tsx     # O coração da aplicação: estado global e lógica de negócios
│   │
│   ├── data/
│   │   └── mock.ts            # Dados de exemplo para popular o banco na primeira execução
│   │
│   ├── services/
│   │   └── geminiService.ts   # Abstração da comunicação com a API Gemini
│   │
│   ├── utils/
│   │   ├── db.ts              # Abstração para o IndexedDB (CRUD, init, seed)
│   │   └── export.ts          # Funções utilitárias para exportar dados
│   │
│   ├── App.tsx                # Componente raiz, orquestra o layout e os modais
│   ├── index.tsx              # Ponto de entrada do React
│   ├── types.ts               # Definições de todas as interfaces TypeScript
│   └── index.html             # Ponto de entrada do HTML, com o importmap
│
├── .env.local                 # Arquivo para variáveis de ambiente (NÃO versionado)
├── package.json
├── vite.config.ts
└── tsconfig.json
```

---

## 6. Executando o Projeto Localmente

Siga os passos abaixo para configurar e rodar a aplicação em seu ambiente de desenvolvimento.

**Pré-requisitos**:
-   [Node.js](https://nodejs.org/) (versão 18 ou superior)
-   `npm` (geralmente instalado com o Node.js)

**Passos**:

1.  **Clone o repositório do projeto**:
    ```bash
    git clone <URL_DO_REPOSITORIO>
    cd <NOME_DO_DIRETORIO>
    ```

2.  **Instale as dependências**:
    ```bash
    npm install
    ```

3.  **Configure a Chave de API do Gemini**:
    -   Crie um arquivo chamado `.env` na raiz do projeto.
    -   Obtenha sua chave de API no [Google AI Studio](https://aistudio.google.com/app/apikey).
    -   Adicione a chave ao arquivo `.env` da seguinte forma:
        ```env
        GEMINI_API_KEY="SUA_CHAVE_DE_API_AQUI"
        ```
    - O `vite.config.ts` está configurado para carregar esta variável e torná-la acessível na aplicação.

4.  **Execute a aplicação em modo de desenvolvimento**:
    ```bash
    npm run dev
    ```

5.  **Acesse no navegador**:
    -   Abra seu navegador e acesse o endereço fornecido pelo Vite (geralmente `http://localhost:5173`).
    -   Na primeira vez que a aplicação for carregada, o IndexedDB será criado e populado com os dados de `data/mock.ts`. Você pode verificar a criação do banco de dados no painel "Application" (ou "Armazenamento") das ferramentas de desenvolvedor do seu navegador.
