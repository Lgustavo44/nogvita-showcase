# Decisões técnicas

Registro curto das escolhas que mais pesaram, com o motivo de cada uma.

## SEO sem renderização no servidor

**Contexto.** As páginas públicas precisam de prévia correta no WhatsApp e no LinkedIn, cujos robôs não executam JavaScript. Pré-renderizar com Angular exigiria duas dependências novas, entrada de servidor e hidratação.

**Decisão.** Um script Node roda depois do `ng build` e gera um HTML por rota pública com título, descrição, Open Graph e canonical, além de `robots.txt` e `sitemap.xml`. Se um marcador esperado não existir, o build falha.

**Efeito.** Prévias corretas sem dependência nova e sem mudar a forma de hospedar. O corpo da página continua vindo do JavaScript.

## CSP estrita no front

**Decisão.** `script-src 'self'`, sem script inline, fonte servida pelo próprio app. Tudo que é externo entra na política de forma explícita e justificada.

**Efeito.** Uma injeção de HTML não consegue carregar script de terceiros.

## Áreas logadas fora do Google

**Decisão.** Cabeçalho `X-Robots-Tag: noindex` para as áreas do paciente, do nutricionista e do admin, e para links com token (confirmação de e-mail, redefinição de senha, verificação de documento).

**Efeito.** Nenhum link pessoal aparece em busca, mesmo que alguém o compartilhe.

## Testes de integração com banco real

**Decisão.** Testcontainers sobe um PostgreSQL 17 por execução da suíte; as migrations rodam antes da API.

**Efeito.** Índices únicos, transações e consultas são testados como em produção, não contra um banco em memória que se comporta diferente.

## Acessibilidade como teste automatizado

**Decisão.** Cada tela passa pelo axe (WCAG 2.2 AA) no Playwright, no desktop, no celular e no tema escuro.

**Efeito.** Contraste, rótulos e foco viram regressão de teste, não revisão manual esquecida.

## Respostas que não revelam contas

**Decisão.** O cadastro responde sempre da mesma forma, exista ou não o e-mail ou o CPF; quem precisa saber recebe a informação por e-mail.

**Efeito.** Ninguém descobre pela tela se uma pessoa usa o NogVita.

## Agendamentos sem servidor sempre ligado

**Decisão.** Um Cloudflare Worker chama a API a cada 15 minutos, autenticado por segredo compartilhado e comparado em tempo constante, para disparar lembretes.

**Efeito.** Lembretes no horário certo sem pagar por um processo agendado permanente.
