---
name: tpklog-harness-ia
description: Montar e manter o "harness" de desenvolvimento com IA do TPKLOG (instruções curtas, documentação viva, portões de qualidade, revisão multiagente, ondas paralelas, memória e handoff, revisão de segurança), inspirado no vibe-coding-toolkit de @matheusgomes. Use quando pedirem para o Claude Code trabalhar melhor no TPKLOG, organizar CLAUDE.md/AGENTS.md, revisar um PR, paralelizar tarefas ou continuar trabalho de outra sessão.
---

# Harness de IA do TPKLOG: o ambiente em que o Claude Code trabalha

Origem: perfil @matheusgomes (Instagram, 613 mil seguidores) e o repositório público dele `github.com/soumatheusgomes/vibe-coding-toolkit` (MIT, ~2,5 mil estrelas, revisão lida em 2026-11-23). A ideia central dele: **"harness engineering"**, ou seja, a qualidade do que a IA entrega depende do ambiente que você prepara para ela: instruções fáceis de achar, contexto confiável, comandos reproduzíveis e **prova** (não "parece pronto").

Esta skill traz o método adaptado ao TPKLOG. Ela **não** instala os plugins de terceiros do toolkit (Ponytail, Caveman, RTK, Graphify, Context7, Superpowers): esses passam pela skill `tpklog-kit-ia` (ler antes, aprovar item a item, escopo de projeto).

## Regras de trabalho

1. **Medir antes de mexer.** Todo número (warnings, arquivos grandes, testes falhando, build) vem de um comando rodado agora, nunca de estimativa ou memória.
2. **Preservar o que existe**: stack, banco, contratos, CLAUDE.md atual, padrões de commit. Migração de stack, troca de ORM, migration em banco, produção e custo novo exigem decisão do usuário.
3. **Nunca enfraquecer proteção para ficar verde**: não desligar teste, não subir limite, não colocar `ignore` genérico.
4. **Sem travas de Git novas** (hooks de pre-commit/pre-push, bloqueio da main) a menos que o usuário peça.
5. **Nada de código do app alterado antes da aprovação.** Esta skill propõe; o usuário decide.
6. Segredos (certificado A1, senhas SEFAZ, tokens de transportadoras/gateways) nunca em instrução, memória, handoff ou relatório.

## Fase 1: Diagnóstico e baseline (somente leitura)

Registre em `docs/setup.md` (crie se não houver equivalente):
- Produto, quem usa (escritório, pátio, motorista em campo, cliente) e fluxos críticos (ex.: emitir CT-e/MDF-e, roteirizar, baixa de entrega com foto/assinatura, rastreamento).
- Stack, versões, gerenciador de pacotes, comandos reais de instalar/rodar/lint/tipos/testes/build.
- Estado do Git e mudanças locais. O que já existe de IA: `CLAUDE.md`, `.claude/` (skills, agents, hooks, settings), `.mcp.json`. Diferencie **instalado**, **configurado** e **funcionando**.
- Rode as checagens seguras que existirem e anote o resultado, inclusive falhas que já existiam antes.

## Fase 2: Instruções curtas e documentação com dono

O agente precisa saber **por onde começar** sem ler o projeto inteiro.

- **`AGENTS.md` curto** (comandos canônicos, acordos de trabalho, mapa para os detalhes). **`CLAUDE.md` vira ponte** que importa o AGENTS.md e guarda só o que é específico do Claude. Se o TPKLOG já tem um CLAUDE.md bom, mantenha-o como fonte e só enxugue.
- **Cada assunto tem um documento dono** (adapte ao que já existe, sem criar fonte concorrente):
  `README.md` (como rodar) · `PRODUCT.md` (problema, público, escopo, critérios de aceite) · `docs/architecture.md` (estrutura, fronteiras, decisões) · `docs/design.md` (interface e acessibilidade) · `docs/onboarding.md` (preparar a máquina) · `docs/quality.md` (checagens e revisão) · `docs/setup.md` (fontes, versões, evidências, pendências).
- **Mudou comportamento, comando, schema ou configuração → atualiza o documento dono no mesmo PR.**
- Acordos a incluir no AGENTS.md: ler código e instruções antes de agir; separar fato, suposição, decisão e evidência; definir critério de aceite e comando de verificação **antes** de implementar; correção de bug com reprodução e teste de regressão quando viável; menor mudança completa (sem abstração para necessidade não pedida); um bloqueio para só o que depende dele.

## Fase 3: Portões de qualidade (ESLint ou equivalente da stack)

- **Teto de 350 linhas por arquivo** como referência (regra `max-lines` ou equivalente). Meça quantos arquivos passam; regra nova em código legado entra como **warn com contagem e plano de redução**; sem violações, entra como **error**.
- Se for TypeScript: avaliar `no-floating-promises` e `no-misused-promises` (promessa esquecida é bug clássico em integração com SEFAZ, gateways e filas).
- Depois de medir, **quebrar arquivos gigantes por responsabilidade real**, em lotes de 3, preservando API pública, tratamento de erro e testes. Cada lote fecha com lint + tipos + testes verdes.
- **Por que o teto importa (reel @matheusgomes, 2026-11-23):** arquivo que cresce a cada prompt chega a milhares de linhas, estoura o contexto do agente e faz a IA alucinar. O teto mantém cada arquivo legível de uma vez só.
- **Ordem do autor, em 3 passos, um de cada vez (esperar terminar antes do próximo):** (1) setup do projeto, (2) instalar os portões de qualidade com o teto de linhas, (3) quebrar os arquivos grandes. A quebra **não é mecânica** ("quebra burra"): separar por módulo, responsabilidade, regra de negócio, classe e função.
- **Trava no commit/push (opcional, só com aprovação):** o autor deixa o lint impedir commit e push de arquivo acima do teto, e a própria IA refatora quando bate na trava. No TPKLOG, isso só entra depois que o código legado estiver dentro do teto (senão trava todo mundo) e com o ok do usuário (regra 4). Alternativa mais leve: a mesma checagem no CI, só para arquivos alterados no PR.
- Um comando único de verificação (`verify`/`check`) que junta lint, tipos, testes e build, sem publicar nada nem rodar migration.
- Stack sem JavaScript usa a ferramenta nativa (não instalar ESLint só para cumprir checklist).

## Fase 4: Revisão multiagente (antes de todo merge não trivial)

1. Congele o alvo: commits base/head exatos (ou snapshot do working tree).
2. Dispare **em paralelo** revisores independentes e **somente leitura**, escolhidos pelo risco do diff:
   - Qualidade geral (legibilidade, erros, código morto, falta de teste).
   - Segurança (OWASP, segredos, injeção, autenticação/autorização, dependências com CVE).
   - Tipagem da linguagem (casts inseguros, async/concorrência).
   - Framework (regras de hooks, renderização, acessibilidade), só se o diff tocar interface.
   - Domínio TPKLOG, quando tocar: cálculo de frete/prazo, status de entrega, documentos fiscais, dados pessoais de motorista/destinatário (LGPD).
3. Cada achado no formato: `arquivo:linha · severidade · afirmação em 1 frase · cenário concreto de falha`. **Achado sem cenário de falha é palpite e é descartado.**
4. Síntese: deduplicar, filtrar (conferindo no código), ordenar CRÍTICO → ALTO → MÉDIO → BAIXO. Lista vazia de CRÍTICO/ALTO é resultado válido.
5. Não corrigir automaticamente: o relatório volta para quem implementa. Rodadas extras só com evidência nova ou pedido do usuário.

## Fase 5: Ondas paralelas e escolha de modelo

- Paralelize só tarefas **independentes**. Cada tarefa leva: objetivo, contexto mínimo, `Files:` (arquivos que pode alterar), `Resources:` (porta, banco de teste, pasta de build) e `Depends-on:`. Campo faltando ou dúvida → tarefa vira serial.
- **Um escritor por arquivo.** Lockfile, configuração compartilhada e commits são do coordenador, que commita tarefa por tarefa depois da onda. Revisores são somente leitura.
- O coordenador confere diff e checagens antes de aceitar o "pronto" de um subagente.
- **Modelo por tarefa**: busca, inventário, leitura de log e mudança mecânica → modelo mais leve; implementação delimitada → intermediário; arquitetura, segurança, dados fiscais, migrations, concorrência → modelo forte desde o início. Não trocar de provedor nem ligar cobrança por API para isso. Limite de tentativas; falhou com evidência, sobe a capacidade em vez de repetir igual.
- Mudança pequena fica com o agente principal: orquestrar tem custo.

## Fase 6: Memória em camadas e handoff

- **Índice curto** carregado sempre (o `MEMORY.md`/CLAUDE.md) + detalhes sob demanda. Critério para salvar: "uma próxima sessão ficaria surpresa e grata de saber isso antes?" (ex.: regra fiscal não óbvia, decisão rejeitada e por quê, bug sutil já pago).
- Cada memória com fonte, data da última verificação e o que a invalida. Remover o que ficou velho.
- **Trabalho inacabado entre sessões** ganha um handoff curto (no plano/tarefa existente): objetivo e aceite · estado conferido (data, commit, mudanças locais) · feito · evidências (comandos e resultados) · falta · bloqueios · recursos em uso (portas, bancos de teste) · o que reconferir. **Ao retomar, conferir arquivos, Git e checagens reais: o handoff é contexto, não prova.**

## Fase 7: Revisão de segurança reutilizável (somente leitura)

- Modo repositório (linha de base) ou modo diff (base/head). Rastrear fronteiras de confiança reais: entradas externas (formulários, webhooks de transportadoras/gateways, importação de XML/planilha), autenticação, autorização por perfil (motorista só vê as próprias entregas), logs sem dados pessoais, segredos.
- Relatório **privado e redigido** (fora do repositório), separando achado confirmado de hipótese. "Nenhum achado" não é garantia de segurança.
- Registrar em `docs/quality.md` como rodar de novo. Não corrigir nem testar em produção sem autorização.

## Fase 8: Avaliar o próprio harness

Depois de mudar instruções, memória ou skills, testar com tarefas descartáveis (em cópia, nunca no projeto real): uma correção de fronteira, um handoff desatualizado, uma revisão que deve ser somente leitura. Anotar passou/falhou/não rodou e o que foi **observado**, não o que o agente disse que fez.

## Fluxo do dia a dia: /autopilot

Para que todo pedido de desenvolvimento rode este método sempre igual (brainstorm, plano, TDD, ciclos de review, PR, merge e limpeza), use a skill `tpklog-autopilot`, que cria o comando `/autopilot` no repositório TPKLOG.

## Saída esperada

1. Tabela por fase: **Já feito** (arquivo:linha) / **Pertinente** (onde, esforço P/M/G) / **Não se aplica** (motivo) / **Conflita** (com o quê).
2. Lista priorizada (primeiro o que reduz retrabalho e risco: AGENTS/CLAUDE enxuto, comando `verify`, revisão multiagente; depois teto de linhas e refatoração).
3. **Parar e pedir aprovação** antes de criar arquivos fora de `.claude/` e `docs/`, e antes de qualquer mudança no código do app.

## Atalho (opcional, só com aprovação)

O autor publica prompts prontos que o Claude Code lê direto do GitHub, por exemplo `docs/prompts/03-multi-agent-code-review.md`, `05-parallel-wave-dispatch.md`, `08-eslint-quality-gates-install.md`, `09-file-size-refactor.md`, `13-security-review.md`. O prompt `12-existing-project-setup.md` faz a adoção completa, mas **instala vários plugins de terceiros e é voltado a Next.js**: no TPKLOG, use-o só como referência ou depois de passar cada componente pela `tpklog-kit-ia`. Fixe a revisão (commit SHA) usada e registre em `docs/setup.md`.

Fora do escopo: o prompt 17 (avaliar o modelo "Jev" da TypeSafe AI) é sobre adotar um modelo pago no produto; só avaliar se o usuário pedir.
