# Quickstart: validar Saúde e Documentos

## Pré-requisitos

Família com um Admin (A), um Membro (B) e um Leitor (C), e um idoso "Maria" (resultado da validação da feature 001).

## Roteiro

1. **Médico (B):** cadastrar "Dra. Ana, Cardiologista" com telefone. Esperado: aparece para A e C; tocar no telefone abre a discagem.
2. **Condição e alergia (B):** registrar "Hipertensão" (ativa) e alergia "Dipirona" (grave). Esperado: ambas aparecem na visão geral e a alergia fica em destaque.
3. **Medicamento (B):** cadastrar "Losartana 50 mg", uso contínuo. Esperado: aparece em "em uso". Cadastrar "Dipirona": aparece o aviso de alergia, sem bloquear.
4. **Documento por foto (B):** fotografar duas páginas de um exame e juntá-las. Esperado: um único documento com duas páginas, categoria Exame.
5. **Busca (A):** buscar por parte do título e filtrar por categoria. Esperado: encontra em menos de 30 segundos.
6. **Receita (B):** anexar receita com validade em 10 dias e vincular a Losartana. Esperado: lembrete "vence em 15 dias" gerado de imediato (já dentro do marco).
7. **Permissão (C):** tentar editar o documento. Esperado: sem botões de edição; consegue visualizar e compartilhar.
8. **Auditoria (A):** abrir o log de acessos a documentos. Esperado: mostra as visualizações de A e C.
9. **Lixeira (B):** excluir o documento e restaurá-lo. Esperado: volta com todas as páginas.
10. **Ficha de emergência (A):** abrir a ficha, ativar modo avião, reabrir. Esperado: abre com data da última sincronização. Gerar PDF e compartilhar.
11. **Limites:** tentar anexar um arquivo de 25 MB. Esperado: bloqueado com a mensagem do limite.

## Testes automatizados de referência

- pgTAP: isolamento entre famílias (tabelas e bucket); Leitor não escreve; idoso arquivado não aceita escrita; `valid_until > issued_on`.
- Jest: validação de arquivo; sugestão de validade da receita; aviso de alergia ignorando acento e caixa.
- Maestro: roteiro 4 e 5.
