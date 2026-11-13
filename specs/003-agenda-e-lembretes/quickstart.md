# Quickstart: validar Agenda e Lembretes

## Pré-requisitos

Família com Admin (A), Membro (B), Leitor (C), idoso "Maria" e o medicamento "Losartana" (feature 002). Dois aparelhos reais ou emuladores com notificações permitidas. Relógio de teste ou horários próximos do momento atual.

## Roteiro

1. **Compromisso (A):** criar consulta para daqui a 3 horas com médico, local e preparo, acompanhante B. Esperado: B é notificado da atribuição; consulta aparece na agenda de todos.
2. **Lembretes de compromisso:** aguardar o lembrete de 2 h antes. Esperado: B (e quem tiver a opção ligada) recebe.
3. **Sem acompanhante (A):** criar consulta daqui a 24 h sem acompanhante. Esperado: Admins e Membros recebem o aviso na hora.
4. **Conflito (A):** criar outro compromisso do mesmo idoso sobreposto. Esperado: aviso de conflito, com opção de continuar.
5. **Posologia (A):** definir Losartana às 08:00 e 20:00 (ajustar um horário para daqui a 2 minutos). Esperado: doses geradas, a próxima é a futura mais próxima.
6. **Confirmar (B):** no horário, tocar em **Tomou** na notificação. Esperado: dose tomada com autor B; feed atualizado; A vê a confirmação.
7. **Concorrência:** com duas doses pendentes iguais, A e B tocam em "Tomou" quase juntos. Esperado: uma confirmação, a outra recebe "já confirmada por…".
8. **Escalonamento:** criar uma dose para daqui a 1 minuto e ignorar. Esperado: lembrete no horário; aos 30 min novo lembrete e aviso aos demais; aos 120 min a dose fica "perdida" (usar relógio simulado para acelerar).
9. **Dose crítica:** marcar o medicamento como crítico, ignorar a dose por 60 min. Esperado: alerta de alta prioridade a todos; com push desativado em B, B recebe e-mail.
10. **Offline:** ativar modo avião em B antes de uma dose. Esperado: a notificação local aparece; confirmar offline e reconectar sincroniza a confirmação.
11. **Permissão desativada:** desativar notificações do app em B. Esperado: aviso persistente no topo ao abrir o app.
12. **Recorrência (A):** criar fisioterapia semanal por 4 semanas, editar a 2ª ocorrência só para ela, cancelar a série. Esperado: ocorrências futuras somem; as passadas ficam.
13. **Leitor (C):** abrir Hoje. Esperado: vê tudo, sem botões de confirmar dose.

## Testes automatizados de referência

- Jest: gerador de doses (horários fixos, intervalo, dias da semana, horário de verão); lógica de escalonamento com relógio simulado; cálculo de conflitos.
- pgTAP: confirmação concorrente; registro tardio (≤ 24 h) e `TOO_LATE`; Leitor não confirma; isolamento entre famílias.
- Integração: motor processa `reminders` idempotentemente (reexecução não duplica).
- Maestro: roteiros 5, 6 e 13.
