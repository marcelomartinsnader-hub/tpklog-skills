# Changelog

## 2026-11-28 (raio-X do roteirizador)
- `tpklog-roteirizador-ia`: corrigido o texto das tentativas antigas (a do quadro já respeitava carga e região) e o "não reaproveitar" vale só para a lógica de sugestão; rota fixa adiada; nova seção "vocação do carro" (permitido = regra dura, fornecedor preferencial e regiões preferenciais em ordem com plano B = regra leve); indicador de constância por carro.

## 2026-11-28 (roteirizador: começar do zero, pesquisa e regras do dono)
- `tpklog-roteirizador-ia`: visão "segundo cérebro" (proativo, explica com dados, debate com o operador); o app é a fonte da verdade (cadastro de carros, tabela de rotas, corredor); inventário do que já existe (inclui Google Maps); conferência de alinhamento (fase 1a); conversa de esclarecimento informal → regras; regras fixas de jornada (utilitário e JF no mesmo dia, pernoite para van/caminhão no interior); médias reais do app (entregas por cidade, pernoite, rendimento ajudante × motorista); ideias da pesquisa de roteirizadores líderes.
- `tpklog-roteirizador-ia`: nova fase 0b para tirar do app as duas tentativas antigas de roteirização automática (inventário antes, canal de teste, produção só com ok do dono), mantendo os dados de cadastro do PERFIL do carro; registro de que o app já tem perfil por carro e das divergências de capacidade.

## 2026-11-28 (pedido do dono: roteirizador automático)
- Nova skill `tpklog-roteirizador-ia`: a IA interpreta as diretrizes do dia (texto ou áudio) em regras estruturadas que o operador confirma; um montador determinístico usa as tabelas de rotas e corredores, frota e escala para montar os carros como rascunho; o operador aprova; ajustes viram sugestões de regra. Base: estudo da planilha de roteirização atual. Fases 0 a 6.

## 2026-11-25 (reel @code.xr DfeTS2Jzcgx, "Glassy Dashboard V24")
- `tpklog-ui-visual`: novos itens 10 (cabeçalho "o que precisa de você hoje" com pendências clicáveis), 11 (ações rápidas com contador) e 12 (card do dia: anel de entregas concluídas + agenda de coletas/entregas); item 7 ganha seletor de período. Vidro fosco e fundo pêssego do reel continuam fora (itens 8 e 9).

## 2026-11-25 (post @laeltecnologia DgDtJi-AK0c)
- Nova skill `tpklog-baixa-whatsapp`: baixa de canhoto por foto no WhatsApp como canal extra ao app, usando a mesma função de baixa. Cascata de identificação configurável por cliente (QR próprio do TPKLOG, código de barras da chave, OCR, pontuação por contexto, escolha do motorista, fila no painel), fase 1 sem auto-aprovar, trilha de auditoria e travas de custo do Google.

## 2026-11-25 (reels de dashboard: @_code_and_chill_ DejdH-cvBKB, @code.xr DfcQ2TrTbGV e DgC6ruhT2NZ)
- Nova skill `tpklog-ui-visual`: padrão visual dos painéis web do gestor (tokens CSS, grid bento, KPIs, quadro de cargas por etapa, modal de detalhe, selo de status em documento, sidebar com badges). Vidro fosco e degradê pastel ficam fora das telas operacionais e do app do motorista. Começa por uma tela piloto.

## 2026-11-23 (aprendizado validado no TPKLOG · v3620)
- `tpklog-padrao-base`: 1º aprendizado validado — prazo em toda chamada de rede em uma peça só (front) + prazo por fetch no backend, com o prazo do cliente sempre maior que o do servidor nas operações fiscais; teste que varre o backend. Item 1 da avaliação de 23/11, aprovado pelo dono e publicado.
- Avaliação completa das skills contra o TPKLOG real (arquivo:linha, 4 classificações) feita no projeto; as skills instaladas lá foram adaptadas à stack (JS puro + Firebase).
## 2026-11-23 (reel @matheusgomes DgBXEBeMEu7, transcrito)
- Nova skill `tpklog-autopilot`: comando `/autopilot` com fluxo padronizado de 10 etapas; merge na main com aprovação por padrão.
- `tpklog-harness-ia` aponta para o `/autopilot`.

## 2026-11-23 (reels @matheusgomes e @othyagoromeiro)
- `tpklog-harness-ia`: motivo do teto de 350 linhas, ordem dos 3 prompts (setup, portões, quebra por responsabilidade) e trava no commit/push como opção a aprovar.
- Reel @othyagoromeiro (31 pedidos de velocidade) conferido item a item: a `tpklog-performance` v2 já cobre os 31, sem mudança.

## 2026-11-23
- Criado o repositório com `tpklog-performance` (v2), `tpklog-ux-motion` e `tpklog-kit-ia`.
- Criada a skill mestra `tpklog-padrao-base`, que consolida as três e recebe os aprendizados validados.
- Nova skill `tpklog-harness-ia` (método do vibe-coding-toolkit de @matheusgomes) e `tpklog-kit-ia` com os candidatos extras do toolkit (Context7, RTK, Ponytail, Caveman, Graphify, agent-browser).
