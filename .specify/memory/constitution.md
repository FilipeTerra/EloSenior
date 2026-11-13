# Constituição do EloSenior

O EloSenior é um aplicativo móvel que centraliza o cuidado de pessoas idosas para as famílias: saúde, médicos, diagnósticos, receitas, agenda, tarefas e despesas. Esta constituição vale para todas as especificações, planos e implementações do projeto. Em caso de conflito, ela prevalece sobre qualquer spec.

## Princípios Fundamentais

### I. Privacidade e LGPD em primeiro lugar (INEGOCIÁVEL)
Dados de saúde são dados pessoais sensíveis (LGPD, art. 5º, II). Todo recurso deve:
- coletar apenas o mínimo necessário e deixar opcional o que não for essencial;
- registrar a autorização de quem cadastra um idoso (consentimento do titular ou do responsável legal) com data e autor;
- isolar os dados por família no nível do banco de dados (RLS), nunca só na interface;
- criptografar dados em trânsito e em repouso, inclusive arquivos;
- permitir exportação e exclusão dos dados pelo titular da conta e pela família, com prazo de carência claro;
- manter trilha de auditoria de acesso e alteração a dados de saúde;
- nunca usar dados de saúde para publicidade nem compartilhá-los com terceiros sem ação explícita do usuário.

### II. Simplicidade para quem já está sobrecarregado
O usuário principal é um familiar cansado, muitas vezes sem pressa para aprender o app. As ações frequentes (registrar dose, ver "o que tem hoje", lançar despesa, anexar documento) devem levar no máximo três toques a partir da tela inicial. Nenhum campo é obrigatório sem necessidade real. Textos em linguagem simples, sem jargão técnico ou médico desnecessário.

### III. Acessibilidade
Conformidade com WCAG 2.1 AA: contraste mínimo 4,5:1, alvos de toque de pelo menos 44×44 pt, suporte a fonte ampliada do sistema (até 200%), leitores de tela (VoiceOver/TalkBack) e nenhuma informação transmitida apenas por cor. Muitos usuários serão eles próprios pessoas 50+.

### IV. Colaboração transparente
Toda criação, edição, exclusão ou confirmação registra autor e data/hora e aparece no feed da família. Ninguém altera dados "em silêncio". Isso reduz conflitos entre familiares e responde à dor de "falhas na comunicação".

### V. Confiabilidade dos lembretes (INEGOCIÁVEL)
Um lembrete de medicamento ou consulta nunca pode falhar em silêncio. O sistema deve: agendar localmente no dispositivo além do push, detectar notificações desativadas e avisar de forma persistente, escalonar para outros membros quando uma dose não for confirmada e registrar o resultado de cada lembrete (entregue, confirmado, perdido).

### VI. Não substituir o profissional de saúde
O app organiza e lembra; não diagnostica, não recomenda doses, não sugere trocas de medicamento. Telas que exibem dados clínicos trazem aviso curto e visível. Funcionalidades de apoio à decisão clínica (interações medicamentosas, IA de leitura de receitas) ficam fora do MVP e exigem revisão específica.

### VII. Idioma e localização
Toda especificação, interface e mensagem em português do Brasil. Moeda BRL, datas dd/mm/aaaa, fuso padrão America/Sao_Paulo (configurável por família). Os termos do domínio seguem `docs/produto/glossario.md`.

### VIII. Testes derivados dos critérios de aceite
Cada cenário Dado/Quando/Então das specs vira pelo menos um teste automatizado. Regras de permissão (RLS) têm testes próprios no banco. Não se considera uma feature pronta sem os testes dos cenários P1 passando.

## Restrições Adicionais

- **Plataforma do MVP:** aplicativo móvel híbrido (iOS e Android). Versão web e painel administrativo ficam fora do MVP.
- **Modelo de conta:** uma família com vários idosos e vários membros. O idoso não tem login no MVP.
- **Papéis:** Admin, Membro e Leitor, conforme `docs/produto/regras-transversais.md`. Permissão por módulo está fora do MVP.
- **Monetização:** fora do MVP. Os limites técnicos (membros, idosos, armazenamento) existem para proteger o serviço, não para cobrar.
- **Funciona mal com internet ruim:** leitura das telas "Hoje", ficha de emergência e lembretes agendados deve funcionar offline.

## Fluxo de Desenvolvimento

1. Método SDD com Spec Kit: `constitution` → `specify` → `clarify` → `plan` → `tasks` → `implement`.
2. Uma feature por branch numerada (`NNN-nome`), uma spec por feature em `specs/NNN-nome/`.
3. Toda spec numera suas regras de negócio como `RN-XXX-NN` e seus requisitos como `FR-NNN`, para rastreabilidade com `docs/produto/mapa-de-features.md`.
4. Decisões assumidas sem confirmação do dono do produto ficam listadas em "Decisões assumidas e pontos em aberto" na própria spec, até serem respondidas.
5. Mudança de comportamento exige atualizar a spec antes do código.

## Governança

Esta constituição substitui outras práticas. Emendas exigem registro da mudança, justificativa e revisão das specs afetadas. Toda revisão de código e toda análise (`/speckit-analyze`) verifica a conformidade com os princípios I e V. Complexidade adicional deve ser justificada na tabela "Complexity Tracking" do plano.

**Versão**: 1.0.0 | **Ratificada**: 2026-11-13 | **Última emenda**: 2026-11-13
