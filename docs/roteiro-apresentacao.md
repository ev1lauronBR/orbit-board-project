# Roteiro de apresentação técnica

Duração alvo: 8 a 12 minutos. Sugestão de divisão de fala entre os integrantes — ajustar conforme a equipe.

1. **Abertura (1 min)**
   Nome do projeto (OrbitBoard) e apresentação dos integrantes da equipe.

2. **Finalidade da aplicação (1 min)**
   Explicar que o OrbitBoard é um painel didático de gestão de projetos, tarefas e equipe, usado como estudo de caso de integração full stack (aplicação fornecida pelo docente).

3. **Arquitetura (2 min)**
   Mostrar o diagrama de `docs/arquitetura.md`: front-end (React + Vite, servido por Nginx), back-end (.NET 8 / ASP.NET Core, dados em memória), comunicação HTTP/JSON, orquestração via Docker Compose.

4. **Demonstração da aplicação funcionando (2 min)**
   - Abrir o dashboard e mostrar as métricas.
   - Criar um projeto e uma tarefa pela interface.
   - Mover uma tarefa entre colunas de status.
   - Mostrar o tratamento de erro (ex.: tentar criar projeto com nome duplicado).

5. **API e Swagger (1-2 min)**
   Abrir `http://localhost:5200/swagger`, mostrar os grupos de endpoints e executar uma chamada de exemplo (`GET /api/dashboard` ou `POST /api/projects`).

6. **Docker Compose (1-2 min)**
   Executar `docker compose up --build`, mostrar os dois containers subindo, as portas expostas (`5200` e `8080`) e as variáveis de ambiente (`.env`, ver `.env.example`).

7. **Testes e evidências (1 min)**
   Apresentar `docs/evidencias-testes.md`: health checks, fluxo de CORS validado entre front-end e back-end, cenários de erro (`400`, `404`, `409`) e logs dos containers.

8. **Ajustes realizados pela equipe (1 min)**
   - CORS tornado configurável para suportar a origem do front-end conteinerizado.
   - Criação dos `Dockerfile` de back-end e front-end e do `docker-compose.yml`.
   - Documentação da pasta `docs/`.

9. **Dificuldades encontradas e como foram resolvidas (1 min)**
   Espaço para a equipe relatar dificuldades específicas enfrentadas durante a execução (preencher com a experiência real da equipe).

10. **Contribuição de cada integrante (1 min)**
    Cada integrante descreve brevemente sua contribuição (revisão de código, testes, documentação, apresentação etc.).
