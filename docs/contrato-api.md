# Contrato da API

Base URL local: `http://localhost:5200`
Swagger/OpenAPI: `http://localhost:5200/swagger`
Health check: `http://localhost:5200/health`

Todas as respostas são em `application/json`. Erros seguem o formato `ProblemDetails` (`application/problem+json`).

## Health

| Método | Endpoint | Descrição | Sucesso |
|---|---|---|---|
| GET | `/health` | Verifica se a API está no ar. | 200 |

```json
{ "status": "healthy", "service": "OrbitBoard.Api", "utcTime": "2026-07-29T10:00:00Z" }
```

## Dashboard

| Método | Endpoint | Descrição | Sucesso |
|---|---|---|---|
| GET | `/api/dashboard` | Métricas consolidadas e tarefas recentes. | 200 |

## Projetos

| Método | Endpoint | Descrição | Sucesso | Erros |
|---|---|---|---|---|
| GET | `/api/projects` | Lista todos os projetos. | 200 | — |
| GET | `/api/projects/{id}` | Consulta um projeto. | 200 | 404 |
| POST | `/api/projects` | Cria um projeto. | 201 | 400, 409 |
| PUT | `/api/projects/{id}` | Atualiza um projeto. | 200 | 400, 404, 409 |
| DELETE | `/api/projects/{id}` | Exclui um projeto sem tarefas vinculadas. | 204 | 404, 409 |

Exemplo de criação (`POST /api/projects`):

```json
{
  "name": "Portal de Aprendizagem",
  "description": "Plataforma para organizar trilhas e conteúdos.",
  "status": "Planning",
  "startDate": "2026-07-29",
  "dueDate": "2026-09-30",
  "ownerId": "GUID_DO_INTEGRANTE"
}
```

Erro de validação (`400`, nome com menos de 3 caracteres):

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Name": ["The field Name must be a string with a minimum length of 3 and a maximum length of 80."]
  }
}
```

Erro de conflito (`409`, nome de projeto já existente):

```json
{
  "status": 409,
  "title": "Conflito de regra",
  "detail": "Já existe um projeto com esse nome."
}
```

## Tarefas

| Método | Endpoint | Descrição | Sucesso | Erros |
|---|---|---|---|---|
| GET | `/api/tasks` | Lista e filtra tarefas. | 200 | — |
| GET | `/api/tasks/{id}` | Consulta uma tarefa. | 200 | 404 |
| POST | `/api/tasks` | Cria uma tarefa. | 201 | 400, 404 |
| PUT | `/api/tasks/{id}` | Atualiza uma tarefa. | 200 | 400, 404 |
| PATCH | `/api/tasks/{id}/status` | Altera somente o status. | 200 | 404 |
| DELETE | `/api/tasks/{id}` | Exclui uma tarefa. | 204 | 404 |

Filtros de `GET /api/tasks` (query string): `projectId`, `status`, `priority`, `assigneeId`, `search`.

Exemplo de criação (`POST /api/tasks`):

```json
{
  "projectId": "GUID_DO_PROJETO",
  "title": "Revisar experiência de cadastro",
  "description": "Validar mensagens, navegação e estados de erro.",
  "status": "Backlog",
  "priority": "High",
  "assigneeId": "GUID_DO_INTEGRANTE",
  "dueDate": "2026-08-20",
  "estimatedHours": 8
}
```

Valores válidos:

- `status`: `Backlog`, `InProgress`, `Review`, `Done`.
- `priority`: `Low`, `Medium`, `High`, `Critical`.

## Equipe

| Método | Endpoint | Descrição | Sucesso |
|---|---|---|---|
| GET | `/api/team-members` | Lista os integrantes disponíveis para responsabilidade em projetos/tarefas. | 200 |

## Mapeamento de erros

| Situação | Status HTTP |
|---|---|
| Corpo da requisição inválido (Data Annotations) | 400 |
| Recurso (`projeto`/`tarefa`/`integrante`) inexistente | 404 |
| Regra de negócio violada (nome duplicado, projeto com tarefas) | 409 |
| Erro não tratado | 500 |
