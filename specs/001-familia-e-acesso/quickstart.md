# Quickstart: validar Família e Acesso

Roteiro de validação ponta a ponta, usado como base dos testes Maestro (histórias P1) e dos testes manuais.

## Pré-requisitos

- Projeto Supabase de teste (ou `supabase start` local) com as migrações aplicadas e `seed.sql` carregado.
- Dois dispositivos ou emuladores (A e B).

## Roteiro

1. **Cadastro (A):** criar conta com e-mail e senha, aceitar termos, declarar maioridade. Esperado: tela "Criar família".
2. **Família e idoso (A):** criar "Família Teste" e cadastrar "Maria" (nasc. 1944). Marcar a autorização. Esperado: tela inicial mostra Maria; A é Admin.
3. **Autorização obrigatória (A):** tentar cadastrar outro idoso sem marcar a autorização. Esperado: bloqueado com mensagem (RN-FAM-07).
4. **Convite (A):** gerar convite com papel Membro e copiar o link. Esperado: link com validade de 7 dias.
5. **Entrada (B):** criar conta nova pelo link. Esperado: B entra como Membro e vê Maria.
6. **Papel Leitor (A→B):** A muda B para Leitor. Esperado: B perde os botões de edição imediatamente.
7. **Último Admin (A):** A tenta sair. Esperado: bloqueado (`LAST_ADMIN`).
8. **Convite inválido:** reabrir o mesmo link em um terceiro aparelho. Esperado: "convite já utilizado".
9. **Remoção (A):** A remove B. Esperado: B perde o acesso; itens criados por B continuam com o nome dele.
10. **Arquivar (A):** arquivar Maria. Esperado: perfil somente leitura, fora da lista principal.
11. **Exclusão (A):** pedir exclusão da família digitando o nome. Esperado: "exclusão agendada" com data; cancelar restaura.

## Testes automatizados de referência

- pgTAP: usuário de uma família não lê nenhuma linha de outra (todas as tabelas); último Admin; limites; unicidade de família ativa.
- Maestro: roteiro 1 a 5 como fluxo P1.
