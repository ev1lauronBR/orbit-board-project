# Arquitetura

## Visão geral

OrbitBoard é uma aplicação full stack composta por três camadas: front-end, back-end/API e infraestrutura conteinerizada. Não há banco de dados externo — os dados vivem em memória no processo do back-end, o que mantém o foco didático na integração HTTP/JSON em vez da configuração de persistência.

```text
┌─────────────────────┐        HTTP/JSON        ┌──────────────────────────┐
│  Front-end (React)  │ ───────────────────────▶ │  Back-end (ASP.NET Core) │
│  Nginx :80 (host    │ ◀─────────────────────── │  Kestrel :5200           │
│  8080 no compose)   │                          │  Dados em memória        │
└─────────────────────┘                          └──────────────────────────┘
```

## Front-end

- **React 18 + Vite**, com rotas via `react-router-dom` (`src/App.jsx`).
- `src/api/client.js` centraliza a comunicação HTTP: monta a URL a partir de `VITE_API_URL`, serializa/desserializa JSON e converte respostas de erro (`ProblemDetails`) em mensagens legíveis.
- Páginas (`src/pages`) tratam estados de carregamento, vazio, sucesso e erro de forma explícita.
- Em produção, o build estático (`vite build`) é servido por Nginx (`frontend/nginx.conf`), que também expõe `/health` para checagem de disponibilidade.

## Back-end

- **.NET 8 / ASP.NET Core Web API**, organizada em `Controllers`, `DTOs`, `Models`, `Services` e `Middleware`.
- `WorkspaceService` concentra as regras de negócio e o armazenamento em memória (protegido por `lock` para chamadas concorrentes).
- `ExceptionHandlingMiddleware` converte exceções de negócio (`NotFoundException`, `ConflictException`, `ValidationException`) em respostas `ProblemDetails` com o status HTTP correspondente (404, 409, 400).
- Validação de entrada via Data Annotations nos DTOs de `CreateProjectRequest`, `CreateWorkItemRequest` etc., aplicada automaticamente pelo `[ApiController]`.
- Swagger/OpenAPI habilitado em `/swagger`.

## Infraestrutura e Docker

- `backend/Dockerfile`: build multi-stage (`dotnet publish` na etapa de build, imagem final `aspnet:8.0` apenas com os artefatos publicados).
- `frontend/Dockerfile`: build multi-stage (`npm run build` na etapa de build, imagem final `nginx:alpine` servindo os arquivos estáticos).
- `docker-compose.yml` na raiz sobe os dois serviços em uma rede própria (`orbitboard`), expondo:
  - back-end em `BACKEND_PORT` (padrão `5200`);
  - front-end em `FRONTEND_PORT` (padrão `8080`).
- A URL da API usada pelo front-end é definida em tempo de build via `VITE_API_URL` (variável do Vite, portanto só existe até o `npm run build`; não é lida em runtime pelo container Nginx).

## Ajuste técnico relevante: CORS

A configuração original de CORS permitia apenas a origem do Vite em desenvolvimento (`http://localhost:5173`). Como o front-end conteinerizado passa a responder em `http://localhost:8080`, a política de CORS foi tornada configurável via `appsettings.json` (chave `Cors:AllowedOrigins`), permitindo múltiplas origens simultaneamente:

```json
"Cors": {
  "AllowedOrigins": [
    "http://localhost:5173",
    "http://localhost:8080"
  ]
}
```

Isso evita erro de CORS tanto rodando `npm run dev` localmente quanto acessando a aplicação via `docker compose up`.

## Fluxo de dados (exemplo: criar tarefa)

1. Usuário preenche `TaskForm` e envia o formulário.
2. `frontend/src/pages/TasksPage.jsx` chama `api.tasks.create(data)`.
3. `POST /api/tasks` é validado pelo `[ApiController]` (Data Annotations) e tratado por `TasksController` → `WorkspaceService.CreateWorkItem`.
4. O serviço confere se o projeto e o responsável existem, cria o item em memória e devolve `WorkItemResponse` (201 Created).
5. O front-end recarrega a lista de tarefas e exibe uma notificação de sucesso.
