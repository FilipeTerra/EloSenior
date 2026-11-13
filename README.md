# EloSenior

Aplicativo móvel que centraliza o cuidado de pessoas idosas para as famílias: saúde, médicos, receitas, documentos, agenda, lembretes de medicamentos, tarefas e despesas.

Este repositório contém, por enquanto, a **estrutura de especificações do MVP** (Spec-Driven Development com [GitHub Spec Kit](https://github.com/github/spec-kit)). O código do app e o design virão depois, a partir destas specs.

## Por onde começar

1. [`docs/produto/visao.md`](docs/produto/visao.md): problema, proposta de valor, escopo e métricas.
2. [`docs/produto/glossario.md`](docs/produto/glossario.md): termos do domínio.
3. [`.specify/memory/constitution.md`](.specify/memory/constitution.md): princípios não negociáveis (LGPD, simplicidade, acessibilidade, confiabilidade dos lembretes).
4. [`docs/produto/regras-transversais.md`](docs/produto/regras-transversais.md): papéis e permissões, limites, retenção, offline.
5. [`docs/produto/mapa-de-features.md`](docs/produto/mapa-de-features.md): dependências e rastreabilidade com os canvases.

## Features do MVP

| # | Feature | Resumo |
|---|---|---|
| [001](specs/001-familia-e-acesso/spec.md) | Família e acesso | Conta, família, papéis (Admin, Membro, Leitor), convites, perfis de idosos |
| [002](specs/002-saude-e-documentos/spec.md) | Saúde e documentos | Médicos, condições, alergias, medicamentos, receitas, documentos, ficha de emergência |
| [003](specs/003-agenda-e-lembretes/spec.md) | Agenda e lembretes | Consultas, exames, doses com confirmação e escalonamento, tela "Hoje" |
| [004](specs/004-registro-de-despesas/spec.md) | Registro de despesas | Lançamento, categorias, relatórios e exportação (sem divisão entre familiares) |
| [005](specs/005-tarefas-e-atualizacoes/spec.md) | Tarefas e atualizações | Tarefas, feed de atividades, divisão do cuidado, registro do dia, notificações |

Cada pasta em `specs/` tem `spec.md` (histórias, regras de negócio `RN-*`, requisitos `FR-*`, fluxos em Mermaid), `plan.md`, `data-model.md`, `contracts/api.md` e `quickstart.md`.

## Decisões de produto já tomadas

- Usuário principal: familiar cuidador. O idoso não tem login no MVP.
- Uma família com vários idosos e vários membros.
- Papéis: Admin, Membro, Leitor.
- Finanças: apenas registro de gastos, sem divisão nem saldo.
- Monetização fora do MVP.
- Plataforma: app híbrido (Expo/React Native) com Supabase.

As hipóteses ainda não confirmadas estão em "Decisões assumidas e pontos em aberto" no fim de cada spec e consolidadas em `mapa-de-features.md`.

## Usando o Spec Kit neste projeto

Os comandos ficam em `.claude/skills/` (`/speckit-clarify`, `/speckit-analyze`, `/speckit-tasks`, `/speckit-implement`). Sugestão de próximo passo: rodar `/speckit-clarify` em cada feature para fechar os pontos em aberto e depois `/speckit-tasks`.

## Licença

A definir.
