---
name: tpklog-padrao-base
description: Padrão-base da Transportech consolidando tudo o que já foi aprendido e validado (método de trabalho com IA, performance, UX e motion). Use ao iniciar um projeto novo para ele já nascer no patamar mais avançado, ao revisar se um projeto existente está no padrão, e ao registrar um aprendizado novo aprovado.
---

# Padrão-base Transportech (skill mestra)

Esta skill é o **ponto de partida de todo projeto novo** e o **registro vivo do que aprendemos**. Ela resume as skills temáticas do repositório `tpklog-skills` e cresce a cada aprendizado validado no TPKLOG.

- Detalhe de cada tema: `tpklog-performance`, `tpklog-ux-motion`, `tpklog-kit-ia` (mesmo repositório).
- Regra de ouro: só entra aqui o que **foi aplicado e aprovado** em um projeto real. Ideia nova sem validação fica na skill temática.

## Modo 1 — Projeto novo (nascer no padrão)

1. Pergunte ao usuário: tipo de produto (web, app do motorista, API, integração), stack desejada e quem usa (escritório, pátio, motorista em campo, cliente final).
2. Monte o esqueleto já com os itens da seção "Padrão mínimo" abaixo, adaptados à stack. Liste o que vai criar e peça confirmação antes.
3. Crie o `CLAUDE.md` do projeto com: stack, comandos (instalar, rodar, testar, lint), convenções, e a linha "Seguir a skill tpklog-padrao-base".
4. Instale as skills do repositório em `.claude/skills/`.
5. Termine com um relatório: o que já nasceu no padrão e o que ficou pendente (com motivo).

## Modo 2 — Projeto existente (conferir o padrão)

Percorra o "Padrão mínimo" e classifique cada item: **Já feito** (arquivo:linha) / **Pertinente** (onde e esforço P/M/G) / **Não se aplica** (motivo) / **Conflita** (com o quê). Traga lista priorizada. **Não altere código do app sem aprovação.**

## Padrão mínimo

### Método de trabalho
- Descobrir a stack e o que já existe antes de mexer; listar arquivos que vai alterar e pedir confirmação.
- Testes e lint rodando desde o primeiro dia; nunca desativar teste para ficar verde.
- Uma mudança por commit, mensagem com o motivo; trabalho em branch e PR, sem merge direto.
- `CLAUDE.md` atualizado; ferramentas de IA de terceiros só depois de ler o código, com versão fixada e escopo do projeto (ver `tpklog-kit-ia`).
- Segredos (certificado A1, senhas SEFAZ, tokens de transportadoras) nunca no repositório nem em memória de IA.

### Performance (ver `tpklog-performance`)
- Toda lista paginada (cursor em tabelas grandes) e virtualizada quando longa.
- Busca/autocomplete com debounce ~300 ms; sem chamadas duplicadas.
- Sem N+1; índices nas consultas comuns; selecionar só as colunas necessárias.
- Pool de conexões explícito; timeout (e retry com backoff quando idempotente) em toda API externa.
- Tarefas lentas (SEFAZ, PDF, e-mail/WhatsApp, importações, rotas, webhooks) em fila de jobs.
- Code splitting por rota; estáticos com hash + cache longo + Brotli/GZIP; fontes locais com `font-display: swap`; imagens WebP/AVIF com `loading="lazy"`.
- Observabilidade desde o início: requisições lentas rastreadas, log de queries lentas, Web Vitals de usuários reais.
- Fluxos do motorista testados com CPU 4x lenta e rede "Slow 3G".

### UX e motion (ver `tpklog-ux-motion`)
- Alvos de toque ≥ 48 dp (56+ no app do motorista); ação principal grande, embaixo, ao alcance do polegar.
- Uma ação primária por tela; opções frequentes primeiro.
- Feedback em < 400 ms: estado pressionado, skeleton, UI otimista com rollback, envio em segundo plano.
- Entradas flexíveis (placa, CPF/CNPJ, CEP, telefone) normalizadas internamente.
- Padrões conhecidos (navegação inferior, pull-to-refresh, câmera nativa).
- Animações de 150 a 300 ms, só `transform`/`opacity`, respeitando "reduzir movimento"; legível sob sol, sem depender só de cor.

## Aprendizados validados

Registro do que foi aplicado e aprovado em projeto real. Formato: `AAAA-MM-DD | projeto | aprendizado | evidência (PR/commit) | skill de origem`.

_(vazio: o primeiro aprendizado validado entra aqui)_

## Ciclo de atualização (obrigatório)

Sempre que um item for **aprovado pelo usuário e aplicado** em qualquer projeto:

1. Se for algo novo ou um ajuste do padrão, acrescente ao "Padrão mínimo" (ou corrija o item existente) e registre uma linha em "Aprendizados validados".
2. Se uma recomendação se mostrou errada ou inútil na prática, remova ou corrija aqui e na skill temática, registrando o motivo.
3. Atualize o `CHANGELOG.md` do repositório `tpklog-skills`.
4. Faça commit e push no repositório `tpklog-skills` (mensagem: `aprendizado: <resumo>`), mostrando o diff ao usuário antes do push.
5. Não grave no repositório nada específico de cliente, dados pessoais ou segredos: só o padrão reutilizável.
