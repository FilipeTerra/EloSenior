# Contratos: Família e Acesso

Operações expostas ao app. Acesso simples usa a API REST automática do Supabase sob RLS. Operações com regra atômica usam funções RPC. Erros seguem o formato `{ code, message }` com `code` estável para o app traduzir.

## Autenticação (Supabase Auth)

| Operação | Detalhe |
|---|---|
| Cadastro | `signUp(email, senha)` + `create_profile(full_name, terms_version, adult_declared)` |
| Login | Edge Function `login-guard` (verifica bloqueio) → `signInWithPassword`. Resposta de bloqueio: `423 LOGIN_LOCKED` com `retry_after` |
| Social | `signInWithOAuth(google | apple)` |
| Recuperar senha | `resetPasswordForEmail(email)` — resposta sempre igual (RN-FAM-14) |

## Funções RPC

| Função | Quem pode | Entrada | Saída | Erros |
|---|---|---|---|---|
| `create_family(name)` | usuário sem família | nome | família | `ALREADY_IN_FAMILY` |
| `create_invite(role, invitee_email?)` | Admin | papel | `{ invite_id, link, expires_at }` | `INVITE_LIMIT`, `MEMBER_LIMIT`, `FORBIDDEN` |
| `revoke_invite(invite_id)` | Admin | id | ok | `NOT_FOUND` |
| `preview_invite(token)` | autenticado | token | nome da família, papel, nome de quem convidou | `INVITE_INVALID` (usado, expirado ou revogado) |
| `accept_invite(token)` | autenticado sem família | token | família | `INVITE_INVALID`, `ALREADY_IN_FAMILY`, `MEMBER_LIMIT` |
| `change_member_role(user_id, role)` | Admin | usuário, papel | ok | `LAST_ADMIN`, `FORBIDDEN` |
| `remove_member(user_id)` | Admin | usuário | ok | `LAST_ADMIN`, `FORBIDDEN` |
| `leave_family()` | membro | — | ok | `LAST_ADMIN` |
| `create_elder(data, authorization_confirmed)` | Admin ou Membro | dados | idoso | `AUTHORIZATION_REQUIRED`, `ELDER_LIMIT` |
| `archive_elder(elder_id, reason?)` | Admin | id | ok | `FORBIDDEN` |
| `reactivate_elder(elder_id)` | Admin | id | ok | `ELDER_LIMIT` |
| `request_export()` | Admin | — | `request_id` | `FORBIDDEN` |
| `request_family_deletion(confirm_name)` | Admin | nome da família | ok | `NAME_MISMATCH` |
| `cancel_family_deletion()` | Admin | — | ok | `NOT_SCHEDULED` |
| `delete_account()` | usuário | — | ok | `LAST_ADMIN` |

Cada função grava um evento em `activity_log` (feature 005) dentro da mesma transação.

## Leitura direta (REST sob RLS)

- `GET families`, `GET family_members?select=*,profiles(full_name)`, `GET elders`, `GET emergency_contacts?elder_id=eq.<id>`.
- `PATCH elders?id=eq.<id>` (Admin ou Membro) e `POST/PATCH/DELETE emergency_contacts` (Admin ou Membro).

## Edge Functions

| Função | Gatilho | Função |
|---|---|---|
| `login-guard` | chamada pelo app antes do login | controla `login_attempts` |
| `export-family` | job criado por `request_export` | monta arquivo (JSON + CSV + documentos), envia link por e-mail |
| `purge-family` | `pg_cron` diário | remove famílias com carência vencida |
| `expire-invites` | `pg_cron` a cada hora | marca convites vencidos |

## Deep link

`elosenior://convite/<token>` e `https://elosenior.app/convite/<token>` (link universal; domínio provisório). Abrir o link chama `preview_invite`.
