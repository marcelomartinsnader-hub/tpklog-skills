---
name: tpklog-ui-visual
description: Padrão visual dos painéis web do TPKLOG (gestor/operação) inspirado em dashboards modernos - KPIs, kanban de cargas por etapa, cards de status, sidebar com badges, tokens e grid bento, detalhe em modal, documento com carimbo de status. Use ao criar ou redesenhar painéis, listas de cargas/viagens, telas de fatura/CT-e/comprovante. Não usar para o app do motorista sem adaptar às regras da tpklog-ux-motion.
---

# TPKLOG: padrão visual de painéis

Origem (25/11/2026): três reels de dashboard avaliados juntos.
- @_code_and_chill_ DejdH-cvBKB (CRM "rapport"): kanban por etapa, KPIs, card que expande em modal, documento com carimbo de status.
- @code.xr DfcQ2TrTbGV ("Dashboard V22"): tokens CSS, grid bento, sidebar em pílula, KPIs com mini barras, vidro fosco.
- @code.xr DgC6ruhT2NZ ("Glassy Sidebar V10"): sidebar com item ativo, badges e modo recolhido.

A ideia aproveitável não é copiar o visual, e sim **organizar a operação de transporte em etapas visíveis**, com números grandes e detalhe sem trocar de tela. Complementa a `tpklog-ux-motion` (leis de UX e animação continuam valendo).

## Passo 0: reconhecer antes de mexer

1. O TPKLOG é JS puro sem build: nada de React/Framer Motion/Tailwind novo. Use CSS (custom properties, grid, `transition`, `@starting-style`) e, onde houver suporte, a View Transitions API com fallback sem animação.
2. Levante tokens/estilos existentes (cores, fontes, raios, sombras) e as telas de painel atuais. Liste o que pretende mudar e **peça aprovação antes**; nada de redesenho geral de uma vez.
3. **Tela piloto primeiro:** aplique o padrão numa única tela do painel do gestor, valide com o usuário e só então estenda às outras.
4. Respeite a fila do `PLANO-SKILLS.md`: os itens aprovados desta skill entram no fim da fila, não furam.

## Itens (avaliar um a um: Já feito / Pertinente / Não se aplica / Conflita)

| # | Padrão do reel | Tradução para o TPKLOG | Prioridade |
|---|---|---|---|
| 1 | Kanban de pipeline com colunas coloridas e contador | Quadro de cargas/viagens por etapa (ex.: Programada, Coleta, Em trânsito, Entrega, Ocorrência, Finalizada), com contador e valor/peso total por coluna. Mover card = mudar status, com confirmação e toast "movido para X" + desfazer. | Alta |
| 2 | KPIs no topo com número grande + chip de variação / mini barras | Faixa de 3 a 5 KPIs do gestor (números com `font-variant-numeric: tabular-nums`) (entregas no prazo %, viagens ativas, ocorrências abertas, faturamento do mês) com variação vs. período anterior e, se houver dado, mini barras dos últimos 7 dias. Máximo 5 (Miller). | Alta |
| 2b | Tokens CSS + grid "bento" | Um arquivo de tokens (cores, raios, sombras, espaçamentos, cores de status) e o painel do gestor montado em grid de cards de tamanhos variados (KPIs, quadro de cargas, ocorrências, agenda de coletas). Base para todos os outros itens. | Alta |
| 3 | Card que expande em modal sobre fundo desfocado | Clicar numa carga abre o detalhe em painel/modal sem perder a lista (manter filtro e rolagem). Fechar com Esc e clique fora; foco preso no modal (acessibilidade). | Média |
| 4 | Documento em modal com ações e carimbo "PAID" | Visualização de CT-e/MDF-e/fatura/comprovante com ações claras (Baixar PDF, Copiar link, Marcar como paga/entregue) e selo de status (PAGO, ENTREGUE, CANCELADO) aparecendo com animação curta (≤ 300 ms). | Média |
| 5 | Cards de cor cheia por categoria | Cor por status, **sempre acompanhada de texto/ícone** (nunca só cor). Paleta de status fixa em tokens (`--status-transito`, `--status-ocorrencia`...), contraste AA mínimo. | Média |
| 6 | Sidebar (pílula escura / glassy) + navegação em pílulas | Sidebar do painel web: logo no topo, item ativo com fundo destacado + ícone + rótulo, **badge numérico de pendências** (ocorrências, coletas atrasadas), modo recolhido só com ícones e tooltip. Abrir/recolher segue a `tpklog-ux-motion` (150 a 300 ms, `transform`). No celular ela vira navegação inferior (Jakob); não aplicar no app do motorista. | Média |
| 7 | Gráficos (linha com tooltip, barras por segmento, anel de meta) | Só onde já existe dado confiável. Preferir SVG/CSS ou biblioteca já presente; não adicionar dependência pesada só para gráfico. | Baixa |
| 8 | Fundo em degradê pastel, texto cinza claro e pequeno | **Não adotar** em telas operacionais: baixo contraste, ruim sob sol e em monitor de pátio. No máximo como fundo do painel do gestor, com texto em contraste AA. | Não |
| 9 | Glassmorphism (vidro fosco: `backdrop-filter: blur` + fundo translúcido) | **Não é padrão.** Opcional só em overlays/modais do painel web desktop, com fundo opaco de fallback e contraste AA garantido. **Proibido no app do motorista**: perde contraste no sol e `backdrop-filter` pesa em Android barato (conflita com `tpklog-performance`). | Não (exceto overlay desktop) |

## Regras

- Cores e raios em CSS custom properties num único arquivo de tokens; nada de cor solta no meio do código.
- Animações seguem a `tpklog-ux-motion`: 150 a 300 ms, só `transform`/`opacity`, respeitar `prefers-reduced-motion`.
- Kanban e listas longas: paginação/virtualização da `tpklog-performance`; arrastar precisa de alternativa por botão/menu (teclado e toque).
- App do motorista: só os itens 4 e 5 (selo de status e cor + texto), adaptados a alvos de toque grandes.

- **Não fazer:** animação de abertura do app (ex.: "revelação circular"), que atrasa o uso; nem copiar o código-fonte que autores de reels oferecem por comentário (terceiros, não auditado, e o TPKLOG é JS sem build: o estilo sai com CSS puro).

## Como conferir

- Nenhuma cor, raio ou sombra literal fora do arquivo de tokens (busca por `#`, `rgb(`, `px` de raio/sombra nos CSS das telas).
- Layout sem rolagem horizontal em 360 px de largura.
- Rolagem do painel sem queda de FPS com CPU 4x lenta no DevTools (`backdrop-filter` é o primeiro suspeito).
- Contraste AA nos textos sobre cores de status.

## Saída esperada

1. Tabela com os itens (1 a 9, incluindo 2b) classificados (arquivo:linha quando "Já feito").
2. Para os "Pertinente": tela alvo, esforço P/M/G e um esboço (HTML/CSS) só da tela piloto (itens 2b + 2 + 1) para aprovação.
3. Não alterar código do app sem aprovação; aprovados entram no `PLANO-SKILLS.md`.
