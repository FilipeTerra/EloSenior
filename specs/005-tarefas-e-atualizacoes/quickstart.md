# Quickstart: validar Tarefas e Atualizações

## Pré-requisitos

Família com Admin (A), Membro (B), Membro (D) e Leitor (C), idoso "Maria", e algumas ações das features 001 a 004 já feitas (dose confirmada, despesa lançada, documento anexado).

## Roteiro

1. **Criar tarefa (A):** "Buscar Losartana na farmácia", atribuída a B, prazo hoje. Esperado: B é notificado; aparece na lista.
2. **Eu faço (A):** criar tarefa sem responsável. D toca em "Eu faço". Esperado: D vira responsável; feed mostra.
3. **Concluir (B):** concluir a tarefa do passo 1. Esperado: A é notificado; conclusão no feed. Reabrir em até 24 h funciona; depois não.
4. **Feed (C):** abrir o feed. Esperado: vê dose, despesa, documento, tarefa, com autor e hora, e nenhum botão de edição.
5. **Filtros (A):** filtrar por módulo "Despesas" e por membro B. Esperado: só os eventos correspondentes.
6. **Objeto excluído (A):** excluir a despesa e voltar ao feed. Esperado: o evento original permanece e há um novo "A excluiu...".
7. **Agrupamento (B):** lançar 5 despesas em sequência. Esperado: o feed mostra "B lançou 5 despesas" agrupadas.
8. **Divisão do cuidado (A):** abrir a tela. Esperado: cargas por membro; com 3+ membros e um com mais de 60%, aparece a sugestão neutra.
9. **Reatribuir (A):** passar uma tarefa de B para D. Esperado: D é notificado; carga atualizada.
10. **Registro do dia (B):** escrever "Dormiu bem" com humor "bem". Esperado: aparece no feed e no histórico da Maria; A comenta e B é notificado.
11. **Edição após 24 h:** (simular relógio) tentar editar o registro. Esperado: só dá para acrescentar nota.
12. **Preferências (D):** desativar a categoria "atividades"; manter "doses". Esperado: D deixa de receber eventos, mas segue recebendo lembretes de dose. Tentar desativar "doses": aparece o aviso de risco.
13. **Resumo diário (D):** ativar. Esperado: uma notificação às 19:00 com o resumo.
14. **Idoso arquivado (A):** arquivar Maria. Esperado: tarefas abertas dela canceladas, com eventos no feed.
15. **Membro removido (A):** remover B. Esperado: tarefas de B ficam "em aberto"; Admin avisado.

## Testes automatizados de referência

- pgTAP: `activity_log` imutável; Leitor não cria tarefa/comentário; reabertura só até 24 h; reatribuição só a membro ativo.
- Cobertura: percorre todas as funções `SECURITY DEFINER` e exige evento no feed.
- Jest: agrupamento por janela de 10 min; cálculo de carga e limiar de 60%.
- Maestro: roteiros 1 a 4.
