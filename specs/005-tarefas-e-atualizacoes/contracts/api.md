# Contratos: Tarefas e Atualizações

## REST (sob RLS)

| Recurso | Operações | Observações |
|---|---|---|
| `tasks` | GET (filtros: `status`, `assignee_id`, `elder_id`, `due_on`), POST, PATCH | Mudanças de status e responsável via RPC |
| `daily_notes` | GET (por idoso, período, humor), POST | Edição só nas primeiras 24 h via RPC |
| `comments` | GET `?parent_type=&parent_id=`, POST | Edição só até 15 min |
| `activity_log` | somente via `feed()` | Sem escrita |
| `notification_preferences` | GET, PUT | Contrato em 003 |

## Funções RPC

| Função | Quem pode | Entrada | Saída | Erros |
|---|---|---|---|---|
| `create_task(data)` | Admin/Membro | título obrigatório | tarefa | `ELDER_ARCHIVED` |
| `take_task(task_id)` | Admin/Membro | | tarefa com responsável | `ALREADY_ASSIGNED` |
| `assign_task(task_id, assignee_id)` | Admin/Membro | | tarefa | `NOT_A_MEMBER` |
| `complete_task(task_id)` | responsável, criador ou Admin | | tarefa | `ALREADY_DONE` |
| `reopen_task(task_id)` | responsável ou Admin | | tarefa | `TOO_LATE` (> 24 h) |
| `cancel_task(task_id)` | Admin/Membro | | tarefa | |
| `add_daily_note(elder_id, body, mood?)` | Admin/Membro | | registro | `ELDER_ARCHIVED` |
| `add_note_addendum(daily_note_id, body)` | Admin/Membro | | nota | |
| `add_comment(parent_type, parent_id, body, mentions[]?)` | Admin/Membro | | comentário | `PARENT_REMOVED`, `NOT_A_MEMBER` |
| `remove_comment(comment_id)` | autor ou Admin | | ok | |
| `feed(cursor?, limit=30, elder_id?, module?, actor_id?)` | membro ativo | | eventos agrupados + `next_cursor` | |
| `care_load(from?, to?)` | membro ativo | | lista por membro | |
| `audit_log(from, to, filters?)` | Admin | | eventos de auditoria | `FORBIDDEN` |

## Contrato de eventos (usado por todas as features)

Todas as RPCs das features 001 a 005 chamam `log_activity` com `type` no formato `<objeto>.<ação>`:

| Módulo | Tipos de evento |
|---|---|
| `family` | `member.joined`, `member.left`, `member.removed`, `member.role_changed`, `invite.created`, `invite.revoked`, `elder.created`, `elder.updated`, `elder.archived`, `elder.reactivated` |
| `health` | `document.created`, `document.updated`, `document.deleted`, `doctor.created`, `condition.created`, `allergy.created`, `medication.created`, `medication.ended`, `prescription.created` |
| `agenda` | `appointment.created`, `appointment.rescheduled`, `appointment.cancelled`, `appointment.completed`, `dose.taken`, `dose.skipped`, `dose.missed` |
| `expenses` | `expense.created`, `expense.updated`, `expense.deleted` |
| `tasks` | `task.created`, `task.assigned`, `task.completed`, `task.reopened`, `task.cancelled`, `note.created`, `comment.created` |

Exemplo de `summary`: "Ana confirmou a dose de Losartana de Maria (08:00)". Resumos sempre em PT-BR, texto simples.

## Realtime

Canal `family:<family_id>` com assinatura de inserções em `activity_log`, filtrado por RLS. O app insere o evento no topo do feed e invalida os caches do módulo correspondente.

## Jobs

| Job | Agendamento | Função |
|---|---|---|
| `task-reminders` | diário, 08:00 no fuso da família | Lembrete de tarefas com prazo hoje; aviso aos Admins de tarefas vencidas há 48 h |
| `activity-digest` | diário, 19:00 | Resumo da categoria `activity` para quem ativou o resumo diário |
| `audit-purge` | mensal | Remove auditoria com mais de 24 meses |
