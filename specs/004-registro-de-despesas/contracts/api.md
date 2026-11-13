# Contratos: Registro de Despesas

## REST (sob RLS)

| Recurso | Operações | Observações |
|---|---|---|
| `expenses` | GET (filtros: `spent_on` entre, `elder_id`, `category_id`, `payer`), POST, PATCH; exclusão lógica | `amount_cents` sempre inteiro; erros de validação com `code` estável |
| `expense_categories` | GET, POST, PATCH | Escrita só Admin |
| `expense_history` | GET `?expense_id=` | Somente leitura |
| `expense_links` | GET, POST, DELETE | |

## Funções RPC

| Função | Quem pode | Entrada | Saída | Erros |
|---|---|---|---|---|
| `create_expense(data, links[]?)` | Admin/Membro | valor em centavos, data, categoria, idoso?, pagador, etc. | despesa e flag `possible_duplicate` | `INVALID_AMOUNT`, `FUTURE_DATE`, `ELDER_ARCHIVED`, `PAYER_REQUIRED` |
| `confirm_duplicate_expense(data)` | Admin/Membro | mesmos dados, após aviso | despesa | |
| `restore_expense(id)` | autor ou Admin | | ok | `EXPIRED` |
| `expense_summary(from, to, group_by, filters?)` | membro ativo | | `[{ key, label, total_cents, count }]` | |
| `expense_compare(from, to)` | membro ativo | | `{ current_cents, previous_cents, delta_pct }` | |

## Edge Function

| Função | Entrada | Saída |
|---|---|---|
| `export-expenses` | `from`, `to`, filtros opcionais | `{ url, expires_in: 600 }` para `.csv`; registra na trilha de auditoria (RN-DES-14) |

Formato do CSV (RN-DES-11): cabeçalho `Data;Categoria;Idoso;Pagador;Valor;Descrição;Forma de pagamento;Comprovante`, valores como `1234,56`, datas `dd/mm/aaaa`, UTF-8 com BOM.

## Erros

| code | Significado |
|---|---|
| `INVALID_AMOUNT` | valor fora do intervalo |
| `FUTURE_DATE` | data posterior a hoje no fuso da família |
| `PAYER_REQUIRED` | pagador membro ou "outra pessoa" sem identificação |
| `ELDER_ARCHIVED` | idoso arquivado não recebe novos lançamentos |
| `FORBIDDEN` | papel sem permissão (Leitor ou Membro em despesa de outro) |
