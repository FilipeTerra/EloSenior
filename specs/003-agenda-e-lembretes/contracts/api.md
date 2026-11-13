# Contratos: Agenda e Lembretes

## REST (sob RLS)

| Recurso | Operações | Observações |
|---|---|---|
| `appointments` | GET (filtros por período, idoso, acompanhante, status), POST, PATCH | Remarcar via RPC para gravar o histórico |
| `appointment_series` | GET, POST, PATCH | Alterações propagam conforme escopo (esta ocorrência / esta e próximas / série) |
| `doses` | GET (por período, idoso, status) | Escrita só via RPC |
| `notification_preferences` | GET, PUT | Por usuário |
| `push_devices` | POST (registrar), PATCH (`notifications_enabled`, `last_seen_at`) | O app atualiza a cada abertura |

## Funções RPC

| Função | Quem pode | Entrada | Saída | Erros |
|---|---|---|---|---|
| `create_appointment(data, companion_id?, series_rule?)` | Admin/Membro | | compromisso (e conflitos detectados) | `ELDER_ARCHIVED` |
| `check_conflicts(elder_id, companion_id?, starts_at, duration_min, exclude_id?)` | Admin/Membro | | lista de conflitos | |
| `reschedule_appointment(id, new_starts_at, scope)` | Admin/Membro | `scope`: `this`, `this_and_following` | compromisso | `NOT_SCHEDULED` |
| `complete_appointment(id, post_notes?, document_ids[]?)` | Admin/Membro | | ok | |
| `cancel_appointment(id, scope)` | Admin/Membro | | ok | |
| `set_medication_schedule(medication_id, schedule, responsible_id?, critical?)` | Admin/Membro | posologia | doses futuras recalculadas | `INVALID_SCHEDULE` |
| `confirm_dose(dose_id, taken_at?)` | Admin/Membro | `taken_at` opcional (registro tardio) | dose | `ALREADY_RESOLVED` (com autor e hora), `TOO_LATE` (> 24 h) |
| `skip_dose(dose_id, reason?)` | Admin/Membro | | dose | `ALREADY_RESOLVED` |
| `log_prn_dose(medication_id, taken_at?)` | Admin/Membro | | dose avulsa | |
| `update_stock(medication_id, quantity)` | Admin/Membro | | ok | |
| `adherence(medication_id, from, to)` | membro ativo | | contagens por semana | |
| `register_device(expo_token, platform, notifications_enabled)` | autenticado | | ok | |

Todas gravam em `activity_log` (feature 005) na mesma transação.

## Jobs e Edge Functions

| Função | Agendamento | O que faz |
|---|---|---|
| `dose-generator` | diário, 03:00 no fuso da família | Gera doses dos próximos 14 dias, cria lembretes `t0` |
| `reminder-engine` | a cada minuto | Seleciona `reminders` vencidos, envia push (agrupa por idoso/horário), registra resultado, cria os marcos `t30`, `t60` (crítico), marca dose `missed` aos 120 min |
| `receipt-checker` | a cada 15 min | Consulta recibos do Expo, desativa tokens inválidos, cria e-mail de alerta para doses críticas quando necessário |
| `appointment-watcher` | a cada hora | Cria aviso de "sem acompanhante" em compromissos a menos de 48 h; renova ocorrências das séries |
| `stock-watcher` | diário | Cria aviso de estoque baixo (5 dias) |

## Carga útil das notificações

```json
{
  "title": "Maria · Losartana 50 mg",
  "body": "Hora de tomar às 08:00",
  "data": { "kind": "dose", "dose_ids": ["…"], "elder_id": "…" },
  "categoryId": "dose_actions",
  "channelId": "dose_default | dose_critical"
}
```

Ações da notificação (categoria `dose_actions`): **Tomou** (chama `confirm_dose`), **Pular** (abre confirmação rápida). Opção do usuário "ocultar detalhes" troca título e corpo por "Lembrete de cuidado".
