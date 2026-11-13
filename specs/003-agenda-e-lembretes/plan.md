# Plano de Implementação: Agenda e Lembretes

**Branch**: `003-agenda-e-lembretes` | **Data**: 2026-11-13 | **Spec**: [spec.md](./spec.md)

Base técnica compartilhada: ver `specs/001-familia-e-acesso/plan.md`. Depende dos modelos de `002-saude-e-documentos` (médicos, medicamentos).

## Resumo

Compromissos, doses e lembretes guardados no Postgres. Um **motor de lembretes** em Edge Function, disparado a cada minuto por `pg_cron`, lê a tabela `reminders` vencida, envia push (Expo Push), registra recibos e aplica o escalonamento. O app complementa com notificações locais agendadas, para que lembretes de dose funcionem sem internet e sem depender só do push.

## Contexto Técnico

**Linguagem/Versão**: TypeScript 5.x, SQL
**Dependências adicionais**: `expo-notifications`, `expo-task-manager`/`expo-background-fetch` (reposição das notificações locais), `rrule` (recorrência), `date-fns-tz`, `react-native-calendars`, provedor de e-mail transacional (Resend ou similar)
**Armazenamento**: Postgres (`appointments`, `appointment_series`, `doses`, `reminders`, `notification_preferences`, `push_devices`)
**Testes**: pgTAP (geração de doses, RLS, concorrência de confirmação), Jest (cálculo de posologia, fuso/horário de verão, escalonamento), testes de integração do motor de lembretes com relógio simulado, Maestro (confirmar dose pela notificação e pela tela Hoje)
**Plataforma-alvo**: iOS 16+, Android 10+ (canais de notificação com prioridade alta para doses críticas)
**Metas de desempenho**: motor processa lembretes vencidos em menos de 30 s de atraso; tela "Hoje" abre em menos de 1 s com cache
**Restrições**: limite de 64 notificações locais no iOS; o push não é garantido, então a entrega é registrada e o fallback atua
**Escala/Escopo**: cerca de 8 telas (Hoje, calendário, detalhe de compromisso, novo compromisso, posologia, histórico de adesão, preferências de notificação, aviso de permissão)

## Verificação da Constituição

| Princípio | Como este plano atende | Situação |
|---|---|---|
| I. Privacidade | Conteúdo da notificação mostra apelido do idoso e nome do medicamento, com opção "ocultar detalhes na tela bloqueada" | OK |
| II. Simplicidade | Confirmar dose em 1 toque (ação na notificação e na tela Hoje) | OK |
| III. Acessibilidade | Ações de notificação com rótulos claros; calendário com leitura por lista alternativa | OK |
| IV. Colaboração | Eventos de dose e compromisso no feed | OK |
| V. Confiabilidade dos lembretes | Push + local + e-mail crítico; registro de resultado; aviso persistente; escalonamento (RN-AGE-07 a 13) | OK |
| VI. Não substituir o profissional | RN-AGE-19: o app não sugere horários ou doses | OK |
| VIII. Testes | Relógio simulado nos testes do motor; cenários de horário de verão | OK |

## Decisões Técnicas

1. **Fonte única da verdade da dose:** a linha em `doses` é gerada pelo servidor; o cliente só confirma. Confirmação por função `confirm_dose` com `UPDATE ... WHERE status = 'pending'` para vencer a concorrência (RN-AGE-10).
2. **Horário de parede:** `scheduled_local_time` + `family.timezone` convertidos em `timestamptz` na geração. Testes cobrem mudanças de horário de verão.
3. **Motor de lembretes idempotente:** cada lembrete tem chave única (`dose:<id>:t0`, `dose:<id>:t30`...). Reexecuções não duplicam.
4. **Notificação local como rede de segurança:** o app do responsável agenda localmente as próximas 60 doses; quando o push do servidor chega, a notificação local equivalente é cancelada para não duplicar.
5. **Recibos de push:** o motor consulta os recibos do Expo; tokens inválidos desativam o dispositivo e disparam o aviso persistente na próxima abertura e o e-mail para crítico (RN-AGE-13).
6. **Perdida** definida por rotina do motor aos 120 min (RN-AGE-09).
7. **Recorrência:** série guarda regra RRULE e gera ocorrências materializadas em janela de 90 dias, renovada por rotina.

## Estrutura do Projeto

```text
specs/003-agenda-e-lembretes/
├── plan.md
├── data-model.md
├── quickstart.md
└── contracts/
    └── api.md

app/src/features/agenda/          # calendário, compromissos, recorrência
app/src/features/today/           # tela Hoje
app/src/features/medication-doses/# confirmar, pular, histórico
app/src/features/notifications/   # permissões, locais, preferências
supabase/functions/reminder-engine/   # motor (cron a cada minuto)
supabase/functions/dose-generator/    # gera doses (cron diário)
supabase/migrations/
supabase/tests/
```

## Rastreabilidade

| Spec | Atendido por |
|---|---|
| FR-001 a FR-004, RN-AGE-01 a 04 | `appointments` + histórico de remarcações + job de acompanhante |
| FR-005, FR-006, RN-AGE-05, 06 | `dose-generator` e posologia em `medications.schedule` |
| FR-007, RN-AGE-09 a 11 | `confirm_dose`, `skip_dose`, `log_prn_dose` |
| FR-008 a FR-012, RN-AGE-07, 08, 12, 13 | `reminder-engine`, `push_devices`, e-mail crítico, aviso persistente |
| FR-013 a FR-015 | telas Hoje/calendário, detecção de conflito, `appointment_series` |
| FR-017, FR-018, RN-AGE-16 | visão de adesão, estoque estimado |

## Complexity Tracking

| Complexidade | Por que é necessária | Alternativa mais simples rejeitada porque |
|---|---|---|
| Motor de lembretes próprio + notificação local | Princípio V exige que lembrete de medicamento não falhe em silêncio | Só push do provedor não dá garantia de entrega nem escalonamento |
