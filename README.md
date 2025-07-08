# nMVP

# Documentação da API: Consulta de Dispensa de Licitação

Este documento fornece instruções detalhadas sobre como utilizar a API do Portal Nacional de Contratações Públicas (PNCP) para consultar dados específicos sobre **Dispensa de Licitação**.

A consulta é realizada através do serviço "Consultar Contratações por Data de Publicação", aplicando-se um filtro específico para a modalidade de contratação.

## 1. Endpoint de Consulta

Para buscar as dispensas de licitação, utilize o seguinte endpoint:

- **Método HTTP:** `GET`
- **URL Base:** `https://pncp.gov.br/api/consulta`
- **Endpoint:** `/v1/contratacoes/publicacao`

## 2. Parâmetro Chave: `codigoModalidadeContratacao`

Para filtrar exclusivamente as **Dispensas de Licitação**, é obrigatório o uso do seguinte parâmetro na sua requisição:

- `codigoModalidadeContratacao=8`

Este código é definido na Tabela de Domínio **5.2. Modalidade de Contratação** do manual do PNCP.

## 3. Parâmetros de Requisição (Filtros)

Os seguintes parâmetros podem ser combinados para refinar a busca.

| Campo | Tipo | Obrigatório | Descrição |
| --- | --- | --- | --- |
| `dataInicial` | Data | Sim | Data inicial do período a ser consultado no formato `AAAAMMDD`. |
| `dataFinal` | Data | Sim | Data final do período a ser consultado no formato `AAAAMMDD`. |
| `codigoModalidadeContratacao` | Inteiro | Sim | **Deve ser `8` para Dispensa de Licitação.** |
| `uf` | String | Não | Sigla da Unidade Federativa (ex: `DF`, `SP`). |
| `codigoMunicipioIbge` | String | Não | Código IBGE do Município (ex: `5300108` para Brasília). |
| `cnpj` | String | Não | CNPJ do órgão ou entidade que publicou a contratação. |
| `codigoUnidadeAdministrativa` | String | Não | Código da Unidade Administrativa do órgão. |
| `idUsuario` | Inteiro | Não | Identificador do sistema/usuário que publicou a contratação. |
| `pagina` | Inteiro | Sim | Número da página para obter os dados. A primeira página é `1`. |
| `tamanhoPagina` | Inteiro | Não | Quantidade de registros por página (padrão: 50, máximo: 500). |

## 4. Exemplos de Requisições

### Exemplo 1: Consulta Básica por Período

Busca por todas as dispensas de licitação publicadas em agosto de 2023.

```bash
curl -X 'GET' \
  'https://pncp.gov.br/api/consulta/v1/contratacoes/publicacao?dataInicial=20230801&dataFinal=20230831&codigoModalidadeContratacao=8&pagina=1' \
  -H 'accept: */*'
```

### Exemplo 2: Consulta por Órgão e Estado (UF)

Busca por dispensas de um órgão específico (pelo CNPJ) no estado de São Paulo.

```bash
curl -X 'GET' \
  'https://pncp.gov.br/api/consulta/v1/contratacoes/publicacao?dataInicial=20230101&dataFinal=20231231&codigoModalidadeContratacao=8&cnpj=00059311000126&uf=SP&pagina=1' \
  -H 'accept: */*'
```

## 5. Estrutura da Resposta

A resposta será um objeto JSON contendo informações de paginação e um vetor `data` com os resultados.

### 5.1. Estrutura de Paginação

| Campo | Tipo | Descrição |
| --- | --- | --- |
| `totalRegistros` | Inteiro | Total de registros encontrados para a consulta. |
| `totalPaginas` | Inteiro | Total de páginas para obter todos os registros. |
| `numeroPagina` | Inteiro | O número da página atual retornada. |
| `paginasRestantes` | Inteiro | Total de páginas restantes. |
| `empty` | Booleano | `true` se o atributo `data` estiver vazio. |

### 5.2. Estrutura dos Dados (`data`)

Cada objeto dentro do array `data` representa uma dispensa e contém os seguintes campos (lista parcial):

```json
{
  "data": [
    {
      "numeroControlePNCP": "00059311000126-1-000001/2023",
      "numeroCompra": "DISPENSA-001/2023",
      "anoCompra": 2023,
      "processo": "12345.678901/2023-01",
      "modalidadeId": 8,
      "modalidadeNome": "Dispensa de Licitação",
      "situacaoCompraId": 1,
      "situacaoCompraNome": "Divulgada no PNCP",
      "objetoCompra": "Contratação de serviços de limpeza e conservação.",
      "valorTotalEstimado": 50000.00,
      "dataPublicacaoPncp": "2023-08-15T09:00:00",
      "orgaoEntidade": {
        "cnpj": "00059311000126",
        "razaosocial": "NOME DO ORGAO PUBLICO",
        "poderId": "E",
        "esferaId": "F"
      },
      "unidadeOrgao": {
        "codigoUnidade": "194035",
        "nomeUnidade": "NOME DA UNIDADE ADMINISTRATIVA",
        "codigoIbge": 5300108,
        "municipioNome": "Brasília",
        "ufSigla": "DF",
        "ufNome": "Distrito Federal"
      },
      "linkSistemaOrigem": "https://link.para.sistema.de.origem/details"
    }
  ],
  "totalRegistros": 1,
  "totalPaginas": 1,
  "numeroPagina": 1,
  "paginasRestantes": 0,
  "empty": false
}
```

## 6. Paginação

Para navegar pelos resultados, utilize os parâmetros `pagina` e `tamanhoPagina`. A resposta da API sempre informará o `totalPaginas` e o `numeroPagina` atual, permitindo que você construa as próximas requisições de forma iterativa até que `paginasRestantes` seja 0.

## 7. Códigos de Retorno HTTP

| Código HTTP | Mensagem | Significado |
| --- | --- | --- |
| `200` | OK | Requisição bem-sucedida e dados retornados. |
| `204` | No Content | Requisição bem-sucedida, mas nenhum registro foi encontrado. |
| `400` | Bad Request | Erro na requisição, como um formato de data inválido. |
| `422` | Unprocessable Entity | Erro de negócio, como parâmetros obrigatórios ausentes. |
| `500` | Internal Server Error | Erro interno no servidor do PNCP. |

---

## Arquitetura e Implementação do MVP de Consulta

Esta seção descreve a arquitetura da aplicação que consome a API do PNCP, seu fluxo de dados e detalhes de implementação.

### 1. Fluxo de Informações e Arquitetura

O sistema é projetado em uma arquitetura de três camadas (frontend, backend, banco de dados), orquestrada com Docker.

1.  **Interação do Usuário (Frontend):**
    *   O usuário acessa a interface web (aplicação React).
    *   Em uma página de busca, ele insere os filtros desejados, como `dataInicial` e `dataFinal`, e clica em "Buscar".

2.  **Requisição Frontend -> Backend:**
    *   A aplicação React captura os dados do formulário e envia uma requisição HTTP (ex: `GET /api/dispensas?dataInicial=...`) para o seu próprio servidor backend (a aplicação FastAPI).

3.  **Requisição Backend -> API Externa (PNCP):**
    *   O backend em FastAPI recebe a requisição.
    *   Ele atua como um cliente para a API do PNCP, construindo a URL de consulta com os parâmetros recebidos e o `codigoModalidadeContratacao=8` fixo.
    *   O serviço `pncp_service.py` encapsula a lógica de comunicação com a API externa.

4.  **Processamento e Persistência (Backend e Banco de Dados):**
    *   O backend recebe a resposta JSON da API do PNCP.
    *   Os dados são validados, transformados para um modelo interno (`backend/app/models/`) e armazenados no banco de dados PostgreSQL para histórico e otimização de consultas.

5.  **Resposta Backend -> Frontend:**
    *   O backend envia uma resposta JSON para a aplicação React com os resultados da consulta formatados.

6.  **Renderização dos Dados (Frontend):**
    *   A aplicação React recebe os dados, atualiza seu estado e renderiza a tabela de resultados.

### 2. Stack de Tecnologia

*   **Frontend:**
    *   **Framework:** React com TypeScript (`.tsx`).
    *   **Build Tool:** Vite (`vite.config.ts`).
    *   **Comunicação HTTP:** Axios ou Fetch API (`src/api/client.ts`).
    *   **Gerenciamento de Estado:** Zustand ou Context API (`stores/`).

*   **Backend:**
    *   **Framework:** Python com FastAPI.
    *   **Validação de Dados:** Pydantic (`backend/app/schemas/`).
    *   **ORM e Migrations:** SQLAlchemy e Alembic (`backend/app/models/` e `backend/migrations/`).

*   **Banco de Dados:**
    *   **SGBD:** PostgreSQL.

*   **Containerização e Implantação:**
    *   **Containers:** Docker (`Dockerfile` em `frontend/` e `backend/`).
    *   **Orquestração:** Docker Compose (`docker-compose.yml`).
    *   **Servidor Web (Frontend):** Nginx (`frontend/nginx.conf`).

### 3. Preparação para Rodar com Docker

O `docker-compose.yml` define e conecta os serviços da aplicação (`frontend`, `backend`, `db`). Para rodar o sistema, execute o seguinte comando na raiz do projeto:

```bash
docker-compose up --build -d
```

Este comando constrói as imagens, cria e inicia os containers em segundo plano. O Nginx serve a aplicação React e redireciona as chamadas de API para o backend FastAPI.

### 4. Implementação da Interface com Tabela e Exportação XLSX

A implementação ocorre em um componente React, como `frontend/src/pages/entities/Atas.tsx`.

1.  **Componente da Tabela:**
    *   O componente `DataTable.tsx` é usado para exibir os dados recebidos da API do backend.

2.  **Botão de Exportação:**
    *   Um botão "Exportar para XLSX" é adicionado à interface. Para a funcionalidade, a biblioteca `xlsx` (SheetJS) deve ser instalada:
    ```bash
    npm install xlsx
    ```

3.  **Lógica da Função de Exportação:**
    *   A função `onClick` do botão executa a conversão dos dados da tabela (armazenados no estado do componente) para o formato XLSX e inicia o download.

    ```typescript
    // Exemplo de código dentro de um componente React
    import * as XLSX from 'xlsx';

    const handleExport = (dataToExport) => {
      if (!dataToExport || dataToExport.length === 0) {
        console.log("Não há dados para exportar.");
        return;
      }
      
      // 1. Criar uma nova planilha a partir dos dados JSON
      const worksheet = XLSX.utils.json_to_sheet(dataToExport);

      // 2. Criar um novo workbook e adicionar a planilha
      const workbook = XLSX.utils.book_new();
      XLSX.utils.book_append_sheet(workbook, worksheet, "Dispensas");

      // 3. Trigger do download do arquivo .xlsx
      XLSX.writeFile(workbook, "Dispensas_de_Licitacao.xlsx");
    };

    // No JSX:
    // <button onClick={() => handleExport(data)}>Exportar para XLSX</button>
    ```


Funcionalidades Essenciais do MVP:

  - Consulta de dispensas com filtros avançados
  - Visualização em tabela responsiva
  - Exportação para Excel/CSV
  - Dashboard básico com métricas

  Stack Tecnológica Moderna:

  - Frontend: React 18 + TypeScript + MUI + Vite
  - DevOps: Docker + GitHub Actions + Nginx

  Plano de Desenvolvimento (10 semanas):

  1. Sprint 1-2: Setup e autenticação
  2. Sprint 3-4: Core features (consulta e filtros)
  3. Sprint 5-6: UX/UI e exportação
  4. Sprint 7-8: Dashboard e analytics
  5. Sprint 9-10: Polimento e deploy

  Critérios de Validação:

  - MVP funcional em 6 semanas
  - Sistema responsivo e acessível
  - Performance <3s
  - Testes automatizados

  O documento agora serve como guia completo para o desenvolvimento do sistema nLic, com foco em validação rápida e entrega
  incremental.
