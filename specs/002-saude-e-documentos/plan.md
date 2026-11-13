# Plano de Implementação: Saúde e Documentos

**Branch**: `002-saude-e-documentos` | **Data**: 2026-11-13 | **Spec**: [spec.md](./spec.md)

Base técnica compartilhada: ver `specs/001-familia-e-acesso/plan.md`.

## Resumo

Tabelas de saúde com RLS por família, armazenamento privado de documentos no Supabase Storage com URLs assinadas curtas, fila de upload offline no app e ficha de emergência em cache local (SQLite/MMKV via TanStack Query persistido) com geração de PDF no aparelho.

## Contexto Técnico

**Linguagem/Versão**: TypeScript 5.x, SQL
**Dependências adicionais**: `expo-image-picker`, `expo-document-picker`, `expo-image-manipulator` (HEIC→JPEG, compressão), `expo-file-system`, `expo-print` (PDF da ficha), `expo-sharing`, `@tanstack/query-async-storage-persister`
**Armazenamento**: Postgres + bucket privado `documents` (caminho `<family_id>/<elder_id>/<document_id>/<arquivo>`)
**Testes**: pgTAP (RLS, lixeira, vínculos), Jest (validação de arquivo, regras de receita, aviso de alergia), Maestro (adicionar documento por foto)
**Restrições**: limite de 20 MB por arquivo; 1 GB por família; URLs assinadas de até 1 hora; trilha de acesso a documentos
**Escala/Escopo**: cerca de 8 telas (lista de documentos, detalhe, adicionar, médicos, condição, alergia, medicamento, ficha de emergência)

## Verificação da Constituição

| Princípio | Como este plano atende | Situação |
|---|---|---|
| I. Privacidade | Bucket privado, política de Storage por `family_id`, URL assinada curta, auditoria de acesso (RN-SAU-14) | OK |
| II. Simplicidade | Botão único "+" com câmera primeiro; campos mínimos: título e categoria | OK |
| III. Acessibilidade | Lista de documentos com texto alternativo; ações por botões grandes | OK |
| IV. Colaboração | Gatilhos gravam no feed (feature 005) | OK |
| V. Lembretes | Avisos de receita usam a mesma infraestrutura da feature 003 | OK |
| VI. Não substituir o profissional | Aviso nas telas clínicas; sem interação medicamentosa (RN-SAU-15) | OK |
| VIII. Testes | Cada RN-SAU tem teste associado | OK |

## Decisões Técnicas

1. **Auditoria de acesso** passa por uma função RPC `get_document_url(document_id)` que grava o acesso e devolve a URL assinada. O app nunca pede URL direta ao Storage.
2. **Cálculo de "em uso"** (RN-SAU-06) por visão SQL `active_medications`, evitando divergência entre telas.
3. **Aviso de receita** gerado por job diário (`pg_cron`) que insere linhas em `reminders` (feature 003), com chaves idempotentes para não duplicar.
4. **Aviso de alergia** no cliente, com normalização `unaccent` + minúsculas, sem consulta a bases externas.
5. **Cache da ficha de emergência:** consulta persistida no aparelho com carimbo da última sincronização, invalidada a cada alteração via feed.

## Estrutura do Projeto

```text
specs/002-saude-e-documentos/
├── plan.md
├── data-model.md
├── quickstart.md
└── contracts/
    └── api.md

app/src/features/health/      # médicos, condições, alergias, medicamentos
app/src/features/documents/   # upload, lista, busca, fila offline
app/src/features/emergency/   # ficha e PDF
supabase/migrations/          # tabelas + RLS + storage policies
supabase/tests/               # pgTAP
```

## Rastreabilidade

| Spec | Atendido por |
|---|---|
| FR-001 a FR-004, RN-SAU-12 | `documents`, `document_files`, fila offline, filtros |
| FR-005, FR-006, RN-SAU-03 | `doctors`, `elder_doctors` |
| FR-007 a FR-010 | `conditions`, `allergies`, `medications` |
| FR-011, FR-012, RN-SAU-08 a 10 | `prescriptions`, `prescription_medications`, job de aviso |
| FR-014, RN-SAU-13 | tela de emergência + `expo-print` |
| FR-017, RN-SAU-14 | `get_document_url`, `document_access_log` |

## Complexity Tracking

Nenhuma violação da constituição.
