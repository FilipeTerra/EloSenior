# Design do MVP

Protótipo navegável em 10 telas de celular (390×844), derivado das specs em `specs/`. Os arquivos em `prototipo/` são o código-fonte do canvas (formato `.dc.html` + `canvas.json`), publicado como artefato no Claude. Para editar, abra o artefato; estes arquivos servem de versionamento.

## Telas e specs de origem

| Tela | Arquivo | Spec |
|---|---|---|
| Hoje (confirmar dose em 1 toque, tarefas, consultas) | `Main.dc.html` | 003 H2 e H3, 005 H1 |
| Agenda (acompanhante, preparo, conflito) | `Agenda.dc.html` | 003 H1 e H4 |
| Saúde do idoso | `Saude.dc.html` | 002 H3, H4 e H6 |
| Documentos (busca e filtros) | `Documentos.dc.html` | 002 H1 |
| Ficha de emergência (offline) | `Ficha.dc.html` | 002 H5 |
| Despesas (total, categorias, lançamentos) | `Despesas.dc.html` | 004 H2 e H3 |
| Nova despesa | `NovaDespesa.dc.html` | 004 H1 |
| Família (feed e divisão do cuidado) | `Familia.dc.html` | 005 H2 e H3 |
| Convite e cadastro | `Convite.dc.html` | 001 H2 |
| Lembrete de dose e escalonamento | `Notificacao.dc.html` | 003 H2, RN-AGE-07 a 13 |

## Decisões visuais

- **Público 50+:** texto mínimo de 14 px (corpo 16–17 px), alvos de toque de 44 a 56 px, contraste de pelo menos 4,5:1, informação nunca só por cor.
- **Tipografia:** Atkinson Hyperlegible (corpo, desenhada para legibilidade) e Fraunces (títulos e horários).
- **Paleta:** índigo `#3D3489` (do Value Proposition Canvas) sobre fundo quente `#FAF7F2`; âmbar `#8A4B08` para atenção; verde `#1F6B45` para confirmado; vermelho `#A8231B` só para alergia grave e alerta crítico.
- **Navegação:** barra inferior com Hoje, Agenda, Saúde, Despesas e Família.

## O que é interativo no protótipo

Confirmar doses e concluir tarefa (Hoje), "Eu acompanho" (Agenda), busca e filtros (Documentos), seleção de categoria, idoso e pagador (Nova despesa), alternar Atividade e Divisão do cuidado (Família) e a navegação entre telas. O restante é visual.

## Fora do protótipo

Relatórios detalhados de despesas, formulários de consulta e medicamento, lixeira, preferências de notificação e onboarding de criação da família.
