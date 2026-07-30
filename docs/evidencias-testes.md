# Evidências de testes

Testes manuais de integração executados localmente, com o back-end e o front-end rodando via `docker compose up --build` (portas padrão: back-end `5200`, front-end `8080`).

## 1. Build e subida dos containers

```text
$ docker compose up --build -d
 Image orbit-board-project-backend Built
 Image orbit-board-project-frontend Built
 Network orbit-board-project_orbitboard Created
 Container orbit-board-project-backend-1 Started
 Container orbit-board-project-frontend-1 Started

$ docker compose ps
NAME                             SERVICE    STATUS         PORTS
orbit-board-project-backend-1    backend    Up             0.0.0.0:5200->5200/tcp
orbit-board-project-frontend-1   frontend   Up             0.0.0.0:8080->80/tcp
```

## 2. Health checks

```text
$ curl http://localhost:5200/health
{"status":"healthy","service":"OrbitBoard.Api","utcTime":"2026-07-29T10:03:29Z"}

$ curl http://localhost:8080/health
{"status":"healthy","service":"orbitboard-frontend"}
```

## 3. Front-end consumindo a API real (via navegador)

Acessando `http://localhost:8080` (front-end conteinerizado), o dashboard carregou os dados vindos do back-end em `http://localhost:5200` sem erros de console. Requisições capturadas na aba de rede do navegador:

```text
OPTIONS http://localhost:5200/api/dashboard  → 204 No Content   (preflight de CORS)
GET     http://localhost:5200/api/dashboard  → 200 OK
OPTIONS http://localhost:5200/api/projects   → 204 No Content
GET     http://localhost:5200/api/projects   → 200 OK
OPTIONS http://localhost:5200/api/tasks      → 204 No Content
GET     http://localhost:5200/api/tasks      → 200 OK
OPTIONS http://localhost:5200/api/team-members → 204 No Content
GET     http://localhost:5200/api/team-members → 200 OK
```

Isso confirma que o ajuste de CORS (múltiplas origens permitidas, ver `docs/arquitetura.md`) funciona corretamente entre a origem `http://localhost:8080` (front-end) e a API em `http://localhost:5200`.

Telas renderizadas corretamente com dados reais da API: Dashboard, Projetos e Tarefas — cada uma exibindo os registros de exemplo carregados na inicialização do back-end (3 projetos, 5 tarefas, 4 integrantes).

## 4. Swagger / documentação dos endpoints

`http://localhost:5200/swagger` carrega a UI do Swagger com todos os grupos de endpoints documentados: `Dashboard`, `Health`, `Projects`, `Tasks`, `TeamMembers`, além dos schemas de request/response.

## 5. Fluxo funcional completo (criar → listar → alterar status → excluir)

Testado manualmente pela interface:

1. Criação de projeto válido → sucesso, projeto listado imediatamente.
2. Criação de projeto com nome duplicado → erro `409` exibido na interface.
3. Criação de tarefa vinculada a um projeto → sucesso, tarefa aparece na coluna "Backlog".
4. Alteração de status da tarefa pelo seletor "Mover para" → tarefa migra de coluna.
5. Exclusão de tarefa → confirmação via `window.confirm`, remoção da lista.
6. Tentativa de excluir projeto com tarefas vinculadas → erro `409` ("O projeto possui tarefas e não pode ser excluído.").

## 6. Testes de erro via linha de comando (Postman/curl)

```text
$ curl -i http://localhost:5200/api/projects/00000000-0000-0000-0000-000000000000
HTTP/1.1 404 Not Found

$ curl -X POST http://localhost:5200/api/projects -H "Content-Type: application/json" \
  -d '{"name":"a","description":"desc","ownerId":"00000000-0000-0000-0000-000000000000"}'
HTTP/1.1 400 Bad Request
{
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": ["The field Name must be a string with a minimum length of 3 and a maximum length of 80."],
    "Description": ["The field Description must be a string with a minimum length of 10 and a maximum length of 500."]
  }
}

$ curl -X POST http://localhost:5200/api/projects -H "Content-Type: application/json" \
  -d '{"name":"Portal de Aprendizagem","description":"Descrição de teste válida","ownerId":"<GUID_VALIDO>"}'
HTTP/1.1 409 Conflict
```

## 7. Logs dos containers

```text
backend-1  | info: Microsoft.Hosting.Lifetime[14]
backend-1  |       Now listening on: http://[::]:5200
backend-1  | info: Microsoft.Hosting.Lifetime[0]
backend-1  |       Application started. Press Ctrl+C to shut down.
backend-1  | info: Microsoft.Hosting.Lifetime[0]
backend-1  |       Hosting environment: Production

frontend-1 | nginx/1.27.5
frontend-1 | start worker processes
frontend-1 | 172.19.0.1 - - "GET / HTTP/1.1" 200 505
frontend-1 | 172.19.0.1 - - "GET /assets/index-*.js HTTP/1.1" 200 184640
frontend-1 | 172.19.0.1 - - "GET /assets/index-*.css HTTP/1.1" 200 7761
```

## 8. Erros encontrados e correções aplicadas

| Problema encontrado | Correção aplicada |
|---|---|
| Não havia `Dockerfile` para back-end nem front-end, nem `docker-compose.yml` — a aplicação só rodava fora de containers. | Criados `backend/Dockerfile`, `frontend/Dockerfile` (builds multi-stage) e `docker-compose.yml` na raiz, com portas e variáveis parametrizadas via `.env`. |
| Política de CORS liberava apenas `http://localhost:5173`, então o front-end servido via Nginx em `http://localhost:8080` seria bloqueado pelo navegador ao chamar a API. | CORS tornado configurável (`Cors:AllowedOrigins` em `appsettings.json`), agora aceitando tanto a porta do Vite (dev) quanto a porta do Nginx (Docker). |
| Repositório não continha documentação de arquitetura, contrato de API ou evidências de teste (itens obrigatórios do trabalho). | Criada a pasta `docs/` com `arquitetura.md`, `contrato-api.md`, `evidencias-testes.md` e `roteiro-apresentacao.md`. |

Nenhum bug funcional foi encontrado no domínio da aplicação (CRUD de projetos/tarefas, validações, tratamento de erros) — todos os fluxos testados retornaram os status HTTP e comportamentos esperados.
