# Regras Transversais

Regras que valem para todas as features. Em conflito com uma spec, vale a constituição (`.specify/memory/constitution.md`) e depois este documento.

## 1. Matriz papel × ação

| Ação | Admin | Membro | Leitor |
|---|:-:|:-:|:-:|
| Ver tudo da família (idosos, saúde, agenda, despesas, tarefas, feed) | ✔ | ✔ | ✔ |
| Baixar e compartilhar documentos | ✔ | ✔ | ✔ |
| Exportar despesas em CSV | ✔ | ✔ | ✔ |
| Criar e editar médicos, condições, alergias, medicamentos, receitas, documentos | ✔ | ✔ | ✘ |
| Criar e editar compromissos, posologia | ✔ | ✔ | ✘ |
| Confirmar, pular ou registrar dose | ✔ | ✔ | ✘ |
| Lançar despesa | ✔ | ✔ | ✘ |
| Editar ou excluir despesa **própria** | ✔ | ✔ | ✘ |
| Editar ou excluir despesa **de outro membro** | ✔ | ✘ | ✘ |
| Criar, atribuir e concluir tarefas; registro do dia; comentar | ✔ | ✔ | ✘ |
| Editar dados e contatos de emergência do idoso | ✔ | ✔ | ✘ |
| Cadastrar idoso | ✔ | ✔ | ✘ |
| Arquivar e reativar idoso | ✔ | ✘ | ✘ |
| Criar e revogar convites; definir papéis | ✔ | ✘ | ✘ |
| Remover membros; transferir administração | ✔ | ✘ | ✘ |
| Gerenciar categorias de despesas | ✔ | ✘ | ✘ |
| Ver trilha de auditoria e uso de armazenamento | ✔ | ✘ | ✘ |
| Exportar dados da família; agendar exclusão da família | ✔ | ✘ | ✘ |
| Sair da família | ✔ (se não for o último Admin) | ✔ | ✔ |
| Excluir a própria conta | ✔ (se não for o último Admin) | ✔ | ✔ |
| Restaurar itens da lixeira | ✔ | ✔ (o que criou) | ✘ |

Permissão por módulo (ex.: esconder finanças do Leitor) está **fora do MVP**.

## 2. Estados do idoso e efeitos

| Estado | Leitura | Escrita | Lembretes |
|---|---|---|---|
| Ativo | tudo | conforme o papel | ativos |
| Arquivado | tudo (somente leitura) | bloqueada | cancelados; tarefas abertas canceladas |

Reativar não recria lembretes antigos.

## 3. Lixeira e exclusão

- Itens excluídos (documentos, médicos arquivados, condições, alergias, medicamentos, despesas, tarefas) vão para a lixeira por **30 dias** e depois são removidos definitivamente, incluindo arquivos.
- Médico com vínculos e categoria de despesa com lançamentos nunca são excluídos, só arquivados.
- Eventos do feed e da auditoria nunca são excluídos por usuários.

## 4. Autoria e histórico

- Toda criação e alteração registra autor e data/hora (`created_by`, `updated_by`).
- Toda ação relevante gera evento no feed (contrato em `specs/005-tarefas-e-atualizacoes/contracts/api.md`).
- Quando um membro sai ou é removido, a autoria continua com o nome. Quando exclui a conta, passa a "Ex-membro".

## 5. Privacidade, retenção e auditoria

| Item | Regra |
|---|---|
| Residência dos dados | Região Brasil (`sa-east-1`) |
| Isolamento | RLS por `family_id` em todas as tabelas e no armazenamento de arquivos |
| Dados do idoso | Mínimo necessário; CPF e cartão SUS opcionais |
| Autorização | Declaração registrada ao cadastrar o idoso (versão do texto, autor, data) |
| Feed | Exibe os últimos 12 meses |
| Trilha de auditoria | Retida por 24 meses; só Admin lê |
| Exclusão da família | 30 dias de carência; remoção definitiva depois; backups expurgados em até 90 dias |
| Exportação | Só Admin; link temporário de 7 dias |
| Links de arquivos | Assinados, validade de até 1 hora |
| Uso para publicidade | Proibido |

Prazos de retenção e texto de consentimento precisam de **validação jurídica** antes do lançamento público.

## 6. Notificações

- Categorias: dose, escalonamento de dose, compromisso, receita, estoque, tarefa, atividade, menção.
- O autor de uma ação não é notificado dela por padrão.
- Lembretes de dose e compromisso têm prioridade sobre qualquer outra categoria.
- Desativar lembretes de dose exige confirmação e deixa aviso persistente no app.
- Opção "ocultar detalhes na tela bloqueada" troca o conteúdo por texto genérico.

## 7. Localização e formatos

| Item | Padrão |
|---|---|
| Idioma | Português do Brasil |
| Fuso | `America/Sao_Paulo`, configurável por família |
| Datas | dd/mm/aaaa; horas em 24 h |
| Moeda | Real (BRL), valores em centavos no banco |
| CSV | `;` como separador, vírgula decimal, UTF-8 com BOM |

## 8. Limites técnicos do MVP

| Limite | Valor |
|---|---|
| Membros por família | 10 |
| Idosos ativos por família | 5 |
| Convites pendentes | 10 |
| Contatos de emergência por idoso | 3 |
| Arquivo | 20 MB (JPEG, PNG, HEIC, PDF) |
| Armazenamento por família | 1 GB |
| Valor de despesa | R$ 0,01 a R$ 9.999.999,99 |
| Ocorrências de uma série de compromissos | 52 |
| Horizonte de geração de doses | 14 dias |

Os limites existem para proteger o serviço, não para cobrar (monetização está fora do MVP).

## 9. Funcionamento offline

| Recurso | Offline |
|---|---|
| Tela "Hoje" e ficha de emergência | Leitura com dados da última sincronização e aviso |
| Lembretes de dose | Notificações locais já agendadas funcionam |
| Confirmar dose | Fica na fila local e sincroniza ao reconectar |
| Anexar documento | Fila local com reenvio automático |
| Demais criações e edições | Exigem conexão no MVP |

## 10. Aviso de responsabilidade

Texto curto exibido nas telas clínicas e na ficha de emergência:

> O EloSenior ajuda a organizar informações e lembretes. Ele não substitui a orientação de um profissional de saúde.

## 11. Fora do MVP (todas as features)

Assinatura e cobrança; divisão e acerto de despesas entre familiares; login ou modo simplificado para o idoso; papel de cuidador profissional; parcerias com instituições; biblioteca de conteúdos; OCR e leitura automática de receitas; verificação de interações medicamentosas; sinais vitais estruturados; permissão por módulo; versão web; link público de emergência; despesas previstas e recorrentes; recorrência de tarefas; várias famílias por usuário.
