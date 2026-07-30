# OrbitBoard

Painel didático de gestão de projetos, tarefas e equipe — aplicação full stack usada como estudo de caso do Módulo 5 (Integração Full Stack) da capacitação em IA e Transformação Digital.

## Equipe

- Thiago Serra Andrade Leite
- Thalia Rodrigues Pinto
- Pamela do Nascimento Bernardo

## Objetivo didático

Aplicar, na prática, os conteúdos de integração full stack: revisar e executar uma aplicação com front-end, back-end/API e infraestrutura conteinerizada, validar a comunicação HTTP/JSON, ajustar CORS e variáveis de ambiente, documentar o contrato da API e registrar evidências de testes.

## Arquitetura resumida

- **Front-end**: React 18 + Vite, servido em produção por Nginx.
- **Back-end**: API REST em .NET 8 / ASP.NET Core, com dados mantidos em memória (sem banco de dados externo).
- **Infraestrutura**: containers Docker para front-end e back-end, orquestrados por `docker-compose.yml`.

Detalhamento completo em [`docs/arquitetura.md`](docs/arquitetura.md).

## Tecnologias utilizadas

| Camada | Tecnologias |
|---|---|
| Front-end | React 18, Vite, React Router, Nginx (produção) |
| Back-end | .NET 8, ASP.NET Core Web API, Swashbuckle (Swagger) |
| Infraestrutura | Docker, Docker Compose |

## Como executar

### Opção 1 — Docker Compose (recomendado)

Pré-requisito: Docker instalado.

```bash
cp .env.example .env
docker compose up --build
```

- Front-end: http://localhost:8080
- Back-end/API: http://localhost:5200
- Swagger: http://localhost:5200/swagger
- Health check da API: http://localhost:5200/health
- Health check do front-end: http://localhost:8080/health

Para encerrar:

```bash
docker compose down
```

### Opção 2 — Executando localmente (sem Docker)

Back-end (a partir da pasta `backend`):

```bash
dotnet restore OrbitBoard.Api.sln
dotnet run --project OrbitBoard.Api
```

A API sobe em `http://localhost:5200`. Detalhes em [`backend/README.md`](backend/README.md).

Front-end (a partir da pasta `frontend`, em outro terminal):

```bash
cp .env.example .env
npm install
npm run dev
```

A aplicação sobe em `http://localhost:5173`. Detalhes em [`frontend/README.md`](frontend/README.md).

## Endpoints principais

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/api/dashboard` | Métricas e tarefas recentes |
| GET/POST | `/api/projects` | Listar / criar projetos |
| GET/PUT/DELETE | `/api/projects/{id}` | Consultar / atualizar / excluir projeto |
| GET/POST | `/api/tasks` | Listar (com filtros) / criar tarefas |
| GET/PUT/DELETE | `/api/tasks/{id}` | Consultar / atualizar / excluir tarefa |
| PATCH | `/api/tasks/{id}/status` | Alterar apenas o status da tarefa |
| GET | `/api/team-members` | Listar integrantes disponíveis |
| GET | `/health` | Health check da API |

Contrato completo, exemplos de request/response e mapeamento de erros em [`docs/contrato-api.md`](docs/contrato-api.md).

## Variáveis de ambiente

Definidas em `.env` na raiz (ver [`.env.example`](.env.example)), usadas pelo `docker-compose.yml`:

| Variável | Padrão | Descrição |
|---|---|---|
| `BACKEND_PORT` | `5200` | Porta do host mapeada para a API. |
| `FRONTEND_PORT` | `8080` | Porta do host mapeada para o front-end (Nginx). |
| `VITE_API_URL` | `http://localhost:5200` | URL da API usada pelo front-end (definida em tempo de build). |
| `ASPNETCORE_ENVIRONMENT` | `Production` | Ambiente do ASP.NET Core dentro do container. |

Para execução local sem Docker, o front-end usa `frontend/.env` (ver [`frontend/.env.example`](frontend/.env.example)).

## Evidências e documentação

- [`docs/arquitetura.md`](docs/arquitetura.md) — arquitetura detalhada e ajustes técnicos realizados.
- [`docs/contrato-api.md`](docs/contrato-api.md) — contrato completo da API.
- [`docs/evidencias-testes.md`](docs/evidencias-testes.md) — testes manuais de integração e evidências de funcionamento.
- [`docs/roteiro-apresentacao.md`](docs/roteiro-apresentacao.md) — roteiro da apresentação técnica final.

## Estrutura do repositório

```text
orbit-board-project/
├── backend/         API .NET 8 (Dockerfile, README.md)
├── frontend/         Aplicação React (Dockerfile, README.md)
├── docs/             Arquitetura, contrato de API, evidências e roteiro de apresentação
├── docker-compose.yml
├── .env.example
├── .gitignore
├── instrucoes.md     Sequência de comandos Git para o repositório da equipe
└── README.md
```

## Contribuição da equipe

> Preencher com a contribuição resumida de cada integrante (ex.: revisão de back-end, ajustes de front-end, Docker, testes, documentação, apresentação).

- Integrante 1 — - Thiago Serra Andrade Leite — back-end / API
- Integrante 2 — Thalia Rodrigues Pinto - Slides apresentação 
- Integrante 3 — Pamela do Nascimento Bernardo - testes / documentação
