---
name: tpklog-autopilot
description: Criar e usar no TPKLOG um comando /autopilot que roda sempre o mesmo fluxo de 10 etapas para qualquer pedido de desenvolvimento (brainstorm, plano, TDD, ciclos de code review, branch, commits, PR, review no PR, merge, limpeza). Use quando pedirem para padronizar como a IA e a equipe desenvolvem, montar o /autopilot, ou quando o usuário iniciar um pedido com /autopilot.
---

# /autopilot do TPKLOG: um pedido, sempre o mesmo processo

Origem: reel de @matheusgomes (21/11/2026, DgBXEBeMEu7), transcrito com Whisper. A ideia dele: em vez de pedir à IA cada etapa ("faz brainstorm", "planeja", "testa", "revisa"...), você cria **uma skill que já conhece o projeto** e dispara o processo inteiro. Quem desenvolve (você ou alguém da equipe) só digita `/autopilot <pedido>`, e o padrão roda igual para todos, porque a skill fica versionada no Git do projeto.

O fluxo dele: brainstorm → planejamento → execução com TDD → **3 a 5 ciclos de code review** → branch e commits certos → PR → **mais 2 a 3 ciclos de review no PR** → merge na main → pull/push na ordem certa → apagar worktrees e branches antigas. Hooks (notificação, teste de banco etc.) entram na mesma skill.

Esta skill tem duas partes: **Parte A** cria o `/autopilot` dentro do repositório TPKLOG (uma vez). **Parte B** é o fluxo que ele executa. Detalhes de revisão multiagente, ondas paralelas e handoff estão na `tpklog-harness-ia`; não repita aqui, referencie.

## Parte A: criar o /autopilot no TPKLOG (uma vez, com aprovação)

1. **Reconhecer o projeto (somente leitura):** stack, comandos reais de instalar/lint/tipos/testes/build, nome da branch principal, padrão de commit em uso (ver `git log`), se existe CI, `gh` CLI instalado e autenticado, testes de banco, hooks e `CLAUDE.md`/`AGENTS.md` existentes.
2. **Mostrar ao usuário** a versão adaptada das 10 etapas (Parte B) com os comandos reais e **as decisões abaixo**, e esperar o ok:
   - Quantos ciclos de review local (padrão: 3; até 5 se o diff tocar dinheiro, fiscal, autenticação ou dados pessoais) e no PR (padrão: 2; até 3).
   - **Merge na main: automático ou com aprovação humana?** Recomendação para o TPKLOG: **com aprovação** até o fluxo provar que é confiável; depois o usuário decide se libera.
   - Padrão de nome de branch (ex.: `feat/`, `fix/`, `chore/`) e de mensagem de commit (ex.: Conventional Commits em português).
   - Hooks/avisos a incluir (ex.: notificação ao terminar, teste de banco local).
3. **Criar a skill com a Skill Creator** (skill oficial, nativa do Claude Code; no Codex também existe) em `.claude/skills/autopilot/SKILL.md` do repositório TPKLOG, com os comandos reais. Mantê-la curta: ela aponta para `AGENTS.md`/`CLAUDE.md` e para as skills do `tpklog-skills` em vez de copiar texto.
4. Acrescentar ao `CLAUDE.md`/`AGENTS.md` uma linha: "Pedidos de desenvolvimento passam por `/autopilot`."
5. **Testar com um pedido pequeno e real** (ex.: validação de CNPJ num formulário) e ajustar a skill com o que falhou. Commitar a skill em branch própria e abrir PR, para valer para toda a equipe.

## Parte B: as 10 etapas que o /autopilot executa

Antes de tudo: `git status` limpo (se houver mudanças locais que não são deste pedido, **parar e perguntar**), `git pull` na branch principal.

1. **Brainstorm:** entender o pedido, o fluxo de negócio afetado (frete, CT-e/MDF-e, coleta, entrega, app do motorista) e o que fica de fora. Uma pergunta por vez, só se mudar decisão importante; o resto vira suposição declarada.
2. **Plano:** critérios de aceite, arquivos que vai tocar, comandos de verificação. Tarefas independentes podem virar ondas paralelas (`tpklog-harness-ia`, Fase 5). Mudança de schema/migration, produção ou custo novo: **parar e pedir aprovação**.
3. **Branch:** criar `tipo/descricao-curta` a partir da main atualizada (worktree opcional, se várias tarefas rodarem ao mesmo tempo).
4. **TDD:** escrever o teste que falha, implementar o mínimo para passar, refatorar. Bug: primeiro o teste que reproduz. Sem teste possível, dizer por quê.
5. **Verificar:** lint + tipos + testes + build com os comandos reais. Nunca desligar teste ou regra para ficar verde.
6. **Code review local (N ciclos):** revisores independentes e somente leitura conforme `tpklog-harness-ia` Fase 4 (cada achado com cenário concreto de falha). Corrigir, reverificar (etapa 5) e repetir até não sobrar CRÍTICO/ALTO ou acabar o número de ciclos combinado. **Parar antes do limite se um ciclo não trouxer achado novo**: ciclos sem evidência nova só gastam tempo.
7. **Commits:** pequenos e com o motivo, no padrão do projeto. Sem segredos (conferir o diff: `.env`, certificado A1, senhas SEFAZ, tokens).
8. **PR:** push da branch e PR com resumo, critérios de aceite, como testar e o que os reviews encontraram/corrigiram.
9. **Review no PR (M ciclos):** revisar o diff final do PR (não só o último commit) e o resultado do CI. Correções voltam para a etapa 5 na mesma branch.
10. **Merge e limpeza:** merge na main **conforme a decisão da Parte A** (com aprovação, por padrão), `git pull` na main local, apagar a branch remota e local **só se já estiver mergeada**, remover worktrees criados por este pedido. Terminar com relatório curto: o que mudou, evidências (comandos e resultados), ciclos de review feitos, pendências.

## Regras que não mudam

- Nunca `push --force` na main, nunca reescrever histórico de branch de outra pessoa, nunca apagar branch não mergeada sem perguntar.
- Falhou o CI ou um teste: diagnosticar e corrigir; após 3 tentativas sem evidência nova, parar e relatar.
- Produção, migrations em banco real, deploy e mensagens para fora (cliente, transportadora, WhatsApp) **nunca** rodam pelo /autopilot sem aprovação explícita.
- Pedido pequeno (texto, ajuste visual simples) pode usar 1 ciclo de review: o fluxo é o mesmo, a intensidade acompanha o risco.

## Avaliação pelo Claude Code

Classificar cada etapa como **Já feito** (o projeto já tem algo equivalente, citar onde) / **Pertinente** / **Não se aplica** / **Conflita**, antes de criar o `/autopilot`. Se o TPKLOG ainda não tem testes ou CI, dizer isso: as etapas 4, 5 e 9 dependem deles, e criar essa base vem primeiro.
