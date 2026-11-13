# Plano de Implementação: Registro de Despesas

**Branch**: `004-registro-de-despesas` | **Data**: 2026-11-13 | **Spec**: [spec.md](./spec.md)

Base técnica compartilhada: `specs/001-familia-e-acesso/plan.md`. Reaproveita o armazenamento de arquivos de `002`.

## Resumo

Tabela `expenses` com valores em centavos, histórico de alterações por gatilho, agregações para relatórios por funções SQL e exportação CSV gerada em Edge Function. O app tem formulário rápido com valores padrão, lista mensal com filtros e relatórios com gráficos simples.

## Contexto Técnico

**Linguagem/Versão**: TypeScript 5.x, SQL
**Dependências adicionais**: `react-native-gifted-charts` (ou `victory-native`) para gráficos, `expo-sharing` para CSV, máscara de moeda (`react-native-currency-input`)
**Armazenamento**: Postgres (`expenses`, `expense_categories`, `expense_history`, `expense_links`); comprovantes no bucket `documents` sob `<family_id>/expenses/<expense_id>/`
**Testes**: pgTAP (RLS, regras de edição por autor/Admin, histórico imutável), Jest (formatação BRL, validação), teste de soma entre lista e relatório, Maestro (lançar despesa)
**Plataforma-alvo**: iOS 16+, Android 10+
**Metas de desempenho**: relatório trimestral de até 5 mil despesas em menos de 2 s
**Restrições**: valores inteiros em centavos (nunca ponto flutuante); fuso da família para agrupar por mês
**Escala/Escopo**: cerca de 5 telas (lista, formulário, detalhe, relatórios, categorias)

## Verificação da Constituição

| Princípio | Como este plano atende | Situação |
|---|---|---|
| I. Privacidade | RLS por família; exportação registrada na auditoria (RN-DES-14); CSV sem links de comprovantes | OK |
| II. Simplicidade | Formulário com valores padrão (hoje, "eu"); teclado numérico; ≤ 3 toques | OK |
| III. Acessibilidade | Valores lidos por leitor de tela em formato falado ("cento e vinte reais"); gráficos com tabela alternativa | OK |
| IV. Colaboração | Histórico por campo e eventos no feed | OK |
| VII. Idioma/localização | Formato BRL, separadores brasileiros, CSV `;` com BOM | OK |
| VIII. Testes | Soma da lista = soma do relatório como teste de propriedade | OK |

## Decisões Técnicas

1. **Centavos como `bigint`** e formatação só na camada de apresentação.
2. **Edição por autor ou Admin** aplicada em RLS (`created_by = auth.uid() OR role = 'admin'`), não só na interface.
3. **Histórico por gatilho** `AFTER UPDATE` que grava campo a campo em `expense_history`.
4. **Agregações** via funções `expense_summary(family_id, from, to, group_by)` com `date_trunc` no fuso da família.
5. **Exportação** por Edge Function que gera CSV em streaming e responde com link assinado de 10 min; registra em auditoria.
6. **Sem tabelas de saldo ou divisão:** a ausência é deliberada (FR-012). Ao evoluir para divisão, adicionar `expense_shares` em nova migração sem alterar `expenses`.

## Estrutura do Projeto

```text
specs/004-registro-de-despesas/
├── plan.md
├── data-model.md
├── quickstart.md
└── contracts/
    └── api.md

app/src/features/expenses/       # lista, formulário, relatórios, categorias
supabase/functions/export-expenses/
supabase/migrations/
supabase/tests/
```

## Rastreabilidade

| Spec | Atendido por |
|---|---|
| FR-001, RN-DES-01, 02 | `expenses` com restrições `CHECK` e validação Zod |
| FR-002, FR-003, RN-DES-09 | `expense_categories` (padrão semeadas por família) |
| FR-005, FR-006, RN-DES-10 | `expense_summary` e filtros |
| FR-007, RN-DES-11 | `export-expenses` |
| FR-008, RN-DES-04 a 06 | RLS + `expense_history` + lixeira |
| FR-010 | `expense_links` |

## Complexity Tracking

Nenhuma violação da constituição.
