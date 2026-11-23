---
name: tpklog-performance
description: Auditoria e otimização de performance do TPKLOG em 5 camadas (tela, rede, servidor, banco, infra) + observabilidade, com forma de conferir cada item. Use quando pedirem para deixar o TPKLOG mais rápido, investigar lentidão, revisar performance ou preparar o app para motorista em celular e rede ruim.
---

# Performance do TPKLOG: auditar, priorizar e corrigir

Origem: dois reels sobre performance de apps feitos com IA, consolidados e adaptados a um sistema de logística/transporte (rastreamento, cargas, coletas e entregas, CT-e/MDF-e/NF-e, integrações com transportadoras, mapas e roteirização, app/tela do motorista em campo).
- v1: @andreias.ai, "22 pedidos de performance por camada".
- v2: @patrickndf, "31 melhorias de performance" (acrescentou virtualização de listas, só as colunas necessárias, pool de conexões, re-renders, cache de estáticos, balanceamento, fontes, monitoramento de usuários reais e teste em celular/rede lenta, além da ideia de **conferir cada item**).

## Regras de trabalho

1. **Primeiro descubra, depois mexa.** Identifique a stack real (framework front, back, ORM, banco, infra, fila, cache) lendo `package.json`, `composer.json`, `requirements.txt`, `*.csproj`, `docker-compose*`, `.env.example` etc. Não assuma tecnologia.
2. **Liste os arquivos que pretende alterar antes de alterar** e peça confirmação ao usuário ao final de cada fase.
3. **Testes têm que continuar passando.** Rode a suíte (ou a parte afetada) antes e depois de cada fase. Nunca desative ou pule teste para ficar verde.
4. **Uma mudança por commit**, com mensagem dizendo o item do checklist (ex.: `perf(db): índice em entregas(status, data_prevista)`).
5. **Só aplique o que é pertinente.** Se um item não se aplica à stack ou já está resolvido, marque "N/A" ou "OK" no relatório com a evidência (arquivo:linha). Não introduza Redis, CDN, OpenTelemetry, RUM, load balancer etc. sem aprovação explícita: proponha, não instale.
6. Mudanças de infraestrutura (CDN, servidor/região, Redis em produção, múltiplas instâncias, pool no banco) são **somente recomendação**, nunca execução.
7. **Todo item tem um "como conferir".** Não marque nada como resolvido sem a medição ou a evidência indicada.

## Fase 1 — Diagnóstico (somente leitura)

Gere `docs/perf/AUDITORIA.md` com uma tabela: item | status (OK / Problema / N/A) | evidência (arquivo:linha) | impacto estimado (alto/médio/baixo) | esforço (P/M/G) | como conferir.

Priorize as telas e endpoints mais usados no dia a dia de logística: listagem/painel de cargas e entregas, rastreamento, busca de cliente/destinatário/NF, mapa de veículos, emissão de documentos fiscais, relatórios e as telas usadas pelo motorista no celular.

### 🖥️ Tela (front-end)
- [ ] Skeleton no lugar de spinner em listas, painéis e detalhes de carga. *Conferir:* rede em "Slow 3G" no DevTools mostra o esqueleto, não tela branca.
- [ ] Imagens (comprovantes, fotos de canhoto, avatares) em WebP/AVIF, com tamanhos definidos. *Conferir:* aba Network mostra `image/webp` ou `avif`; Lighthouse sem alerta de formato.
- [ ] `loading="lazy"` em imagens fora da tela (galeria de canhotos, histórico de ocorrências). *Conferir:* imagens abaixo da dobra só baixam ao rolar.
- [ ] UI otimista: mudanças de status ("saiu para entrega", "entregue") refletem na tela antes da resposta, com rollback em erro. *Conferir:* com a rede lenta, o status muda na hora; simular erro 500 desfaz a mudança e avisa.
- [ ] Respostas de IA (se houver assistente/resumo) em streaming token a token. *Conferir:* primeiro texto aparece em menos de 1 s.
- [ ] **Virtualização de listas longas** (painel de entregas, eventos de rastreamento, posições de GPS, itens de NF). *Conferir:* com 5.000 linhas, o DOM tem só as linhas visíveis e a rolagem fica fluida.
- [ ] **Sem re-renders desnecessários** (memoização de componentes de lista/mapa, estado no nível certo, mapas que não remontam a cada atualização de posição). *Conferir:* React Profiler (ou equivalente) mostra que atualizar um item não re-renderiza a lista toda.
- [ ] **Fontes hospedadas no próprio app**, só os pesos usados, `font-display: swap`. *Conferir:* nenhuma requisição a fonts.googleapis em produção; no máximo 2 ou 3 arquivos de fonte.

### 🌐 Rede
- [ ] Debounce (≈300 ms) em busca/autocomplete (placa, CNPJ, NF, cidade, CEP). *Conferir:* digitar "SAO PAULO" gera 1 requisição, não 9.
- [ ] Sem chamadas de API duplicadas (efeitos que disparam 2x, falta de cache de query no cliente). *Conferir:* abrir cada tela principal e contar requisições iguais na aba Network.
- [ ] Paginação em toda listagem e API (por cursor em tabelas grandes). *Conferir:* nenhum endpoint de lista devolve tudo; a resposta tem `limit`/`cursor`.
- [ ] Estáticos servidos por CDN. *Conferir:* header de resposta indica o CDN.
- [ ] **Compressão Brotli (ou GZIP)** no servidor/CDN. *Conferir:* `content-encoding: br` ou `gzip` em JS, CSS e JSON grandes.
- [ ] **Cache dos estáticos** com nomes com hash e `Cache-Control: public, max-age=31536000, immutable`; HTML sem cache longo. *Conferir:* na segunda visita, JS/CSS vêm "from disk cache".
- [ ] Code splitting por rota/página (mapa, relatórios, módulo fiscal sob demanda). *Conferir:* o bundle inicial não contém a lib de mapas nem a de gráficos.
- [ ] Dependências sem uso ou pesadas removidas. *Conferir:* bundle analyzer antes/depois; `depcheck` (ou equivalente) limpo.
- [ ] Scripts de terceiros (chat, analytics, mapas) carregados por último / `defer` / `async`. *Conferir:* não aparecem bloqueando a renderização no Lighthouse.

### ⚙️ Servidor
- [ ] Tarefas lentas em fila de jobs: emissão e consulta de CT-e/MDF-e/NF-e na SEFAZ, PDF/DACTE, e-mail/WhatsApp, importação de XML/planilhas, cálculo de rotas, webhooks para clientes. *Conferir:* o endpoint responde em menos de 300 ms e devolve um id de job/status.
- [ ] Timeout (e retry com backoff, quando idempotente) em toda API externa: SEFAZ, transportadoras parceiras, gateways, mapas/roteirização, rastreadores/telemetria. *Conferir:* nenhum cliente HTTP sem timeout explícito; simular API externa travada não trava a tela.
- [ ] Requisições independentes em paralelo (cliente, veículo e motorista ao mesmo tempo). *Conferir:* o trace mostra as chamadas sobrepostas, não em escada.
- [ ] Páginas públicas (rastreamento por código para o cliente final) renderizadas no servidor ou no build, quando a stack permitir. *Conferir:* o HTML inicial já traz o conteúdo.
- [ ] **Pool de conexões** com o banco configurado (tamanho adequado ao número de instâncias e ao limite do banco). *Conferir:* configuração explícita no ORM/driver; sob carga, nenhum erro de "too many connections" nem conexão aberta por requisição.

### 🗄️ Banco de dados
- [ ] Fim do N+1 (listar cargas e buscar cliente/motorista/ocorrências em loop). *Conferir:* o nº de queries da listagem é constante, não cresce com o nº de linhas.
- [ ] Índices nas consultas mais comuns: status, datas (coleta/previsão/entrega), cliente, filial, placa, nº de NF/CT-e, chave de acesso. *Conferir:* `EXPLAIN` sem varredura completa nas tabelas grandes.
- [ ] **Só as colunas necessárias** (nada de `SELECT *` ou ORM carregando XML da NF, payload de webhook ou foto em listagens). *Conferir:* queries das listagens selecionam colunas explícitas; tamanho da resposta antes/depois.
- [ ] Cache (Redis ou equivalente) nos dados mais consultados e que mudam pouco: tabelas de frete, cidades/IBGE, CFOP, parâmetros, dashboard agregado. Defina TTL e invalidação. *Conferir:* taxa de acerto do cache e tempo do endpoint antes/depois.

### 🏗️ Infra (somente recomendação)
- [ ] Servidor de aplicação na mesma região/rede do banco. *Conferir:* latência app→banco abaixo de ~5 ms.
- [ ] **Tráfego distribuído entre várias instâncias** (balanceador), com a aplicação sem estado em memória local (sessão, uploads e fila fora da instância). *Conferir:* derrubar uma instância não derruba o sistema.

### 📡 Observabilidade e teste real
- [ ] Rastreamento das requisições lentas (OpenTelemetry ou o APM já existente) nos endpoints críticos e chamadas externas. *Conferir:* dá para ver, por endpoint, onde está o tempo.
- [ ] Log de consultas lentas habilitado no banco.
- [ ] **Monitorar a velocidade dos usuários reais** (RUM / Web Vitals: LCP, INP, CLS), separando escritório e motorista em campo. *Conferir:* painel com p75 por tela.
- [ ] **Testar em celular e rede lenta**, que é a realidade do motorista na estrada. *Conferir:* rodar os fluxos do motorista (ver entregas do dia, registrar ocorrência, enviar foto do canhoto, dar baixa) com CPU 4x mais lenta e rede "Slow 3G" no DevTools, e registrar os tempos.

## Fase 2 — Plano

A partir da auditoria, monte `docs/perf/PLANO.md` ordenado por impacto ÷ esforço. Para cada item: arquivos a alterar, como medir o antes/depois (tempo de resposta, nº de queries, tamanho do bundle, LCP/INP) e risco. **Pare e peça aprovação do usuário.**

## Fase 3 — Execução

Implemente os itens aprovados, um por vez:
1. Meça o antes (use o "como conferir" do item).
2. Liste os arquivos que vai alterar.
3. Altere, rode os testes e o lint.
4. Meça o depois e registre no `PLANO.md`.
5. Commit.

Migrações de índice: gere a migration no padrão do projeto; avise se a tabela é grande (pode travar escrita) e sugira criação concorrente quando o banco suportar.

## Fase 4 — Verificação e relatório

- Rode a suíte completa de testes.
- Atualize `docs/perf/AUDITORIA.md` com o status final e a evidência de cada "como conferir".
- Entregue ao usuário um resumo curto: o que foi feito, ganho medido, o que ficou como recomendação (infra, CDN, Redis, OTel, RUM, balanceamento) e próximos passos.
