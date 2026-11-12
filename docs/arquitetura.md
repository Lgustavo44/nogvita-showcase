# Arquitetura

## Visão geral

```mermaid
flowchart LR
  P[Paciente<br/>celular] --> W[Front Angular<br/>Vercel]
  N[Nutricionista<br/>computador] --> W
  W -->|HTTPS + JWT| A[API .NET 10<br/>Render]
  A --> D[(PostgreSQL<br/>Supabase)]
  A --> E[Resend<br/>e-mail]
  A --> F[Open Food Facts]
  C[Cloudflare Worker<br/>cron 15 min] -->|segredo compartilhado| A
  A -->|Web Push| P
```

## API em camadas

```mermaid
flowchart TB
  Api[NogVita.Api<br/>controllers, filtros, ProblemDetails] --> App[NogVita.Application<br/>casos de uso, validação, regras]
  App --> Dom[NogVita.Domain<br/>entidades e invariantes]
  Infra[NogVita.Infrastructure<br/>EF Core, e-mail, PDF, push] --> App
  Infra --> Dom
```

- **Domain** guarda as regras que não podem ser quebradas: CPF válido, CRN normalizado, estados de acompanhamento.
- **Application** orquestra cada caso de uso e fala com o mundo por interfaces (e-mail, relógio, armazenamento).
- **Infrastructure** implementa essas interfaces: banco, Resend, geração de PDF, notificações web push.
- **Api** expõe os casos de uso e padroniza os erros em ProblemDetails.

## Fluxo de acompanhamento

```mermaid
sequenceDiagram
  participant Pa as Paciente
  participant Api as API
  participant Nu as Nutricionista
  Pa->>Api: cria conta e confirma o e-mail
  Pa->>Api: busca por nome ou CRN e pede acompanhamento
  Api-->>Nu: notificação do pedido
  Nu->>Api: aceita e monta o plano alimentar
  Pa->>Api: marca refeições, água e atividade
  Nu->>Api: vê a adesão dos últimos 7 dias
```

## Front-end

- Rotas carregadas sob demanda; as páginas públicas não pesam no pacote inicial da área logada.
- Estado com signals e `httpResource`; sessão em `sessionStorage` com renovação de token única por vez.
- Páginas públicas recebem HTML por rota gerado no build (título, descrição, Open Graph e canonical), sem renderização no servidor.
