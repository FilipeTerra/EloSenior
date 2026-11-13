# Quickstart: validar Registro de Despesas

## Pré-requisitos

Família com Admin (A), Membro (B), Leitor (C) e dois idosos: "Maria" e "João".

## Roteiro

1. **Lançar (B):** despesa de R$ 87,50, categoria Farmácia, idoso Maria, comprovante por foto. Esperado: salva em menos de 30 s, com data de hoje e pagador B; aparece no feed.
2. **Validações (B):** tentar valor 0, valor negativo e data de amanhã. Esperado: os três bloqueados com mensagem clara.
3. **Duplicidade (B):** repetir a despesa do passo 1. Esperado: aviso de duplicidade; confirmar salva.
4. **Pagador diferente (A):** lançar R$ 300,00 de Consulta de João, pagador "o próprio idoso". Esperado: salva sem calcular dívida.
5. **Lista e total (C):** abrir Despesas. Esperado: total do mês = soma dos lançamentos; C não vê botões de edição.
6. **Filtros (A):** filtrar por idoso Maria, depois por categoria Consulta. Esperado: lista e total refletem o filtro.
7. **Permissão de edição:** B edita a própria despesa (ok); B tenta editar a de A (negado); A edita a de B (ok). Esperado: histórico mostra valor antigo e novo.
8. **Excluir e restaurar (A):** excluir uma despesa. Esperado: sai dos totais; restaurada, volta.
9. **Categoria própria (A):** criar "Cuidadora de fim de semana" e lançar uma despesa nela. Esperado: disponível para todos.
10. **Relatório (A):** abrir Relatórios do trimestre. Esperado: totais por categoria, idoso e pagador conferem com a lista; texto informando que não há cálculo de dívida.
11. **Exportar (A):** exportar CSV e abrir no Excel/Planilhas. Esperado: colunas e acentos corretos; sem links de comprovante.
12. **Idoso arquivado (A):** arquivar João e tentar novo lançamento para ele. Esperado: bloqueado; as despesas antigas continuam nos relatórios.

## Testes automatizados de referência

- Jest: formatação e parsing BRL; validação de valor e data.
- pgTAP: regras RLS de autor/Admin/Leitor; histórico imutável; lixeira fora dos totais.
- Propriedade: soma de `expense_summary` = soma de `expenses` para qualquer filtro.
- Maestro: roteiro 1 e 5.
