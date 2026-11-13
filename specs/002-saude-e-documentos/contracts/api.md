# Contratos: Saúde e Documentos

Leitura e escrita simples via REST do Supabase sob RLS. Operações com regra ou auditoria via RPC.

## REST (sob RLS)

| Recurso | Operações | Observações |
|---|---|---|
| `doctors` | GET, POST, PATCH | Exclusão só se sem vínculos; senão `PATCH status=archived` (RN-SAU-03) |
| `elder_doctors` | GET, POST, DELETE | |
| `conditions`, `allergies` | GET, POST, PATCH; exclusão lógica | |
| `medications` | GET, POST, PATCH; exclusão lógica | `active_medications` para listar em uso |
| `documents` | GET (com filtros), PATCH; exclusão lógica | filtros: `category`, `doctor_id`, `document_date`, busca por texto |
| `prescriptions`, `prescription_medications` | GET, POST, PATCH | validação `valid_until > issued_on` no banco |

## Funções RPC

| Função | Quem pode | Entrada | Saída | Erros |
|---|---|---|---|---|
| `create_document(elder_id, title, category, document_date?, doctor_id?, tags?, files[])` | Admin/Membro | metadados + descrição dos arquivos já enviados ao Storage | documento | `FILE_TOO_LARGE`, `FILE_TYPE_NOT_ALLOWED`, `QUOTA_EXCEEDED`, `ELDER_ARCHIVED` |
| `create_prescription(document_id, kind, issued_on, valid_until, doctor_id?, medication_ids[])` | Admin/Membro | | receita | `INVALID_VALIDITY` |
| `get_document_url(document_id, action)` | membro ativo | `view`, `download` ou `share` | `{ url, expires_in }` (≤ 3600 s) e grava em `document_access_log` | `NOT_FOUND` |
| `restore_item(table, id)` | Admin/Membro | | ok | `NOT_IN_TRASH`, `EXPIRED` |
| `end_medication(medication_id, ends_on, reason?)` | Admin/Membro | | ok | |
| `emergency_sheet(elder_id)` | membro ativo | | JSON com dados da ficha e `generated_at` | |
| `storage_usage()` | Admin | | bytes por idoso e total | |

## Upload

1. O app pede ao Storage um upload no caminho `<family_id>/<elder_id>/<document_id>/<n>.<ext>`.
2. Depois do envio, chama `create_document`, que valida tamanho, tipo MIME e cota.
3. Se estiver offline, o app guarda o arquivo e a chamada em fila local e repete até concluir (idempotente pelo `document_id` gerado no cliente).

## Jobs

| Job | Agendamento | Função |
|---|---|---|
| `prescription-reminders` | diário, 08:00 no fuso da família | cria lembretes de 15 dias, 3 dias e vencida (chave `prescription:<id>:<marco>`) |
| `trash-purge` | diário | remove definitivamente itens da lixeira com mais de 30 dias, inclusive arquivos |
