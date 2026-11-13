# Plano de Implementação: Tarefas e Atualizações

**Branch**: `005-tarefas-e-atualizacoes` | **Data**: 2026-11-13 | **Spec**: [spec.md](./spec.md)

Base técnica compartilhada: `specs/001-familia-e-acesso/plan.md`. Reaproveita push, preferências e motor de lembretes da `003-agenda-e-lembretes`.

## Resumo

Tabelas de tarefas, comentários e registros do dia, mais o `activity_log` imutável que todas as features alimentam por uma função utilitária `log_activity()` chamada dentro das funções RPC (mesma transação). O app mostra o feed com paginação por cursor, a tela de divisão do cuidado com agregações SQL e preferências de notificação por categoria. Atualizações em tempo real do feed usam Supabase Realtime.

## Contexto Técnico

**Linguagem/Versão**: TypeScript 5.x, SQL
**Dependências adicionais**: Supabase Realtime (canal por família), lista virtualizada (`@shopify/flash-list`), `date-fns`
**Armazenamento**: Postgres (`tasks`, `comments`, `daily_notes`, `activity_log`, `audit_log`)
**Testes**: pgTAP (imutabilidade do feed, RLS, reatribuição, tarefas ao arquivar idoso), Jest (agrupamento de eventos, cálculo de carga), teste que garante que cada RPC das features 001 a 004 grava evento, Maestro (criar e concluir tarefa)
**Plataforma-alvo**: iOS 16+, Android 10+
**Metas de desempenho**: 30 primeiros eventos em menos de 1 s; novo evento no feed em até 2 s via Realtime
**Restrições**: o feed não pode ser editado por nenhum papel; auditoria separada e restrita a Admin
**Escala/Escopo**: cerca de 6 telas (tarefas, detalhe de tarefa, feed, divisão do cuidado, registro do dia, preferências)

## Verificação da Constituição

| Princípio | Como este plano atende | Situação |
|---|---|---|
| I. Privacidade | RLS por família; auditoria distinta com retenção definida; notificação com opção de ocultar detalhes | OK |
| II. Simplicidade | "Eu faço" em um toque; criação com título como único campo obrigatório | OK |
| III. Acessibilidade | Feed navegável por leitor de tela, agrupado por dia | OK |
| IV. Colaboração transparente | Esta feature é a implementação do princípio: `log_activity()` em toda alteração | OK |
| V. Confiabilidade dos lembretes | Alerta ao desativar lembretes de dose (RN-TAR-12) | OK |
| VIII. Testes | Teste que percorre todas as RPCs e exige evento correspondente | OK |

## Decisões Técnicas

1. **`log_activity(family_id, type, module, elder_id, object_ref, summary)`** é a única forma de inserir no `activity_log`. Ela é chamada por todas as RPCs; um teste de cobertura lista as funções `SECURITY DEFINER` do schema e falha se alguma não gravar evento.
2. **Agrupamento só na apresentação:** o armazenamento mantém cada evento; a consulta agrupa por (autor, módulo, tipo, janela de 10 min).
3. **Notificações a partir do feed:** um gatilho em `activity_log` cria linhas em `reminders` (`kind = 'task' | 'activity'`) para os destinatários que têm a categoria ligada, reaproveitando o `reminder-engine`. Resumo diário é uma tarefa do motor às 19:00.
4. **Divisão do cuidado** por função `care_load(family_id, from, to)` com contagem em SQL; a lógica do limiar de 60% fica no cliente para ajustar sem migração.
5. **Menções:** texto com `@[uuid]` resolvido no cliente; o servidor extrai os identificadores para notificar.
6. **Auditoria (`audit_log`)** alimentada por gatilhos nas tabelas de saúde e por `get_document_url`; leitura restrita a Admin.

## Estrutura do Projeto

```text
specs/005-tarefas-e-atualizacoes/
├── plan.md
├── data-model.md
├── quickstart.md
└── contracts/
    └── api.md

app/src/features/tasks/
app/src/features/feed/
app/src/features/care-load/
app/src/features/daily-notes/
app/src/features/notifications/    # preferências (compartilhado com 003)
supabase/migrations/
supabase/tests/
```

## Rastreabilidade

| Spec | Atendido por |
|---|---|
| FR-001 a FR-003, RN-TAR-01 a 04 | `tasks`, RPCs de tarefa, lembretes de prazo |
| FR-004 a FR-006, RN-TAR-05 a 07 | `activity_log`, `log_activity()`, consulta com agrupamento |
| FR-007, RN-TAR-08 | `care_load()` + tela |
| FR-008, FR-009, RN-TAR-09, 10 | `daily_notes`, `comments` |
| FR-010 a FR-012, RN-TAR-11, 12 | `notification_preferences` (003) + alerta na UI |
| FR-014, RN-TAR-14 | `audit_log` |

## Complexity Tracking

Nenhuma violação da constituição.
