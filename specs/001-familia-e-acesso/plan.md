# Plano de Implementação: Família e Acesso

**Branch**: `001-familia-e-acesso` | **Data**: 2026-11-13 | **Spec**: [spec.md](./spec.md)

Este plano também define a **base técnica compartilhada** de todo o MVP. Os planos 002 a 005 apenas a referenciam.

## Resumo

Autenticação, família, papéis, convites e perfis de idosos, com isolamento de dados por família garantido por políticas RLS no Postgres. O app móvel consome o Supabase diretamente para leitura e escrita simples, e usa funções de banco (RPC) para operações que exigem regras atômicas (aceitar convite, trocar Admin, remover membro).

## Contexto Técnico

**Linguagem/Versão**: TypeScript 5.x (app e funções de borda), SQL (Postgres 15+)

**Dependências principais**: Expo SDK (React Native), Expo Router, Supabase JS, TanStack Query, Zod, react-hook-form, Expo SecureStore, Expo LocalAuthentication

**Armazenamento**: Supabase (Postgres com RLS, Auth, Storage com bucket privado, Edge Functions, pg_cron)

**Testes**: Jest + React Native Testing Library (unidade/componentes), pgTAP (RLS e funções SQL), Maestro (fluxos ponta a ponta)

**Plataforma-alvo**: iOS 16+ e Android 10+ (build com EAS)

**Tipo de projeto**: app móvel + backend gerenciado (Supabase)

**Metas de desempenho**: telas principais com dados em cache abrem em menos de 1 s; operações de escrita confirmam em menos de 2 s em 4G

**Restrições**: dados de saúde sensíveis (LGPD); funcionamento offline parcial (leitura em cache); região do projeto Supabase no Brasil (`sa-east-1`) para residência de dados

**Escala/Escopo**: dezenas a poucas centenas de famílias no MVP; cerca de 25 telas no total

## Verificação da Constituição

| Princípio | Como este plano atende | Situação |
|---|---|---|
| I. Privacidade e LGPD | RLS em todas as tabelas por `family_id`; consentimento registrado; exportação e exclusão com carência; região `sa-east-1` | OK |
| II. Simplicidade | Onboarding em 3 passos; convite por link compartilhável | OK |
| III. Acessibilidade | Componentes base com tamanho mínimo de toque e fonte dinâmica (definidos na etapa de design) | OK |
| IV. Colaboração transparente | Gatilhos no banco gravam em `activity_log` (feature 005) | OK |
| V. Confiabilidade dos lembretes | Não se aplica a esta feature | n/a |
| VI. Não substituir o profissional | Não se aplica a esta feature | n/a |
| VII. Idioma | i18n com PT-BR como único idioma, chaves de texto centralizadas | OK |
| VIII. Testes | pgTAP cobre cada regra RN-FAM; Maestro cobre histórias P1 | OK |

## Decisões Técnicas (resumo; razões em `research.md` quando virar tarefa)

1. **Supabase em vez de backend próprio:** reduz tempo de MVP; RLS cumpre o isolamento exigido pela constituição. Risco: dependência de fornecedor, mitigado por usar apenas Postgres padrão e migrações em SQL versionadas.
2. **Regras críticas em funções SQL (`SECURITY DEFINER`)** com verificação explícita do papel: `accept_invite`, `change_member_role`, `remove_member`, `leave_family`, `archive_elder`. O app nunca altera `family_members` diretamente.
3. **Convite por token opaco** (32 bytes aleatórios, só o hash fica salvo) com deep link `elosenior://convite/<token>` e link universal.
4. **Bloqueio por tentativas**: o Supabase Auth aplica limites de taxa; o bloqueio de 15 min após 5 falhas é complementado por uma Edge Function de login (ver `contracts/api.md`).
5. **Exportação e exclusão** como jobs assíncronos em Edge Functions agendadas por `pg_cron`.

## Estrutura do Projeto

### Documentação desta feature

```text
specs/001-familia-e-acesso/
├── plan.md
├── data-model.md
├── quickstart.md
└── contracts/
    └── api.md
```

### Código-fonte (raiz do repositório, criado na fase de implementação)

```text
app/                          # Expo (React Native)
├── app/                      # rotas (Expo Router): (auth), (tabs), convite/[token]
├── src/
│   ├── features/
│   │   ├── auth/
│   │   ├── family/
│   │   └── elders/
│   ├── components/           # base acessível compartilhada
│   ├── lib/                  # supabase client, query client, i18n, permissões
│   └── types/                # tipos gerados do banco
└── __tests__/

supabase/
├── migrations/               # SQL versionado (schema, RLS, funções)
├── functions/                # Edge Functions (login-guard, export-family, purge-family)
├── tests/                    # pgTAP
└── seed.sql                  # dados de teste

e2e/                          # fluxos Maestro
```

**Decisão de estrutura:** um único repositório (monorepo simples) com `app/` e `supabase/`. Sem camada de API própria.

## Rastreabilidade

| Spec | Onde é atendido |
|---|---|
| FR-001, FR-014, FR-015 | Supabase Auth + Edge Function `login-guard` |
| FR-003 a FR-013 | Tabelas e funções em `data-model.md` e `contracts/api.md` |
| FR-016 | Expo LocalAuthentication no cliente |
| FR-017, FR-018 | Edge Functions `export-family`, `purge-family`, função `delete_account` |
| RN-FAM-01 a 16 | Restrições, funções SQL e políticas RLS, cada uma com teste pgTAP |

## Complexity Tracking

Nenhuma violação da constituição. Sem itens a justificar.
