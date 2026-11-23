---
name: tpklog-ux-motion
description: Audita e melhora telas do TPKLOG aplicando leis de UX (Fitts, Hick, Miller, Doherty, Postel, Jakob) e princípios de animação/microinteração de forma sóbria. Use ao criar ou revisar telas, formulários, listas, botões, feedbacks de carregamento ou animações no app TPKLOG.
---

# TPKLOG: leis de UX e microinterações

Origem: reel @devai.ana (14/11/2026), "O Claude entende 20 princípios de animação e UX pelo nome". A ideia central é útil: nomear o princípio no prompt gera interface melhor. Esta skill adapta a lista ao contexto do TPKLOG (logística e transporte: motoristas em campo, operadores de pátio/expedição, gestores em painel).

## Passo 0: reconhecer o projeto antes de agir

1. Identifique a stack de front-end (React, React Native, Flutter, Vue, etc.), a biblioteca de animação já usada (Framer Motion, Reanimated, Animated, CSS transitions, Lottie) e o design system/tokens existentes.
2. Mapeie os tipos de usuário e telas: app do motorista (mobile, uso em movimento, sol, luva), telas operacionais (pátio, coleta, entrega, ocorrências, comprovante/POD) e painéis web (rastreamento, KPIs).
3. Não adicione dependência nova de animação se já existir uma. Se não existir nenhuma, prefira transições nativas da plataforma ou CSS antes de propor biblioteca.
4. Liste o que encontrou e o que pretende mudar ANTES de editar. Mudanças grandes de layout precisam de confirmação do usuário.

## Parte 1: leis de UX (prioridade alta)

Para cada tela revisada, verifique e corrija:

| Lei | O que checar no TPKLOG |
|---|---|
| **Fitts** / áreas de clique maiores | Alvos de toque mínimos de 48x48 dp (ideal 56+ no app do motorista). Ações principais (Iniciar viagem, Confirmar entrega, Registrar ocorrência) grandes e ao alcance do polegar, na parte inferior. Espaçamento entre ações destrutivas e principais. |
| **Hick** | Poucas opções por decisão. Listas de motivos de ocorrência agrupadas ou com os mais frequentes primeiro. Uma ação primária por tela. |
| **Miller** | Agrupar dados em blocos (ex.: NF, destinatário, endereço, janela de entrega). Códigos longos (chave de NF-e, rastreio, placa) exibidos em grupos legíveis. |
| **Doherty (resposta < 400 ms)** | Toda ação dá feedback imediato: estado de pressionado, skeleton em listas, otimismo na UI quando seguro, indicador de envio em segundo plano para fotos/assinaturas. Considerar conexão ruim e modo offline. |
| **Postel** | Aceitar entradas flexíveis: placa com ou sem hífen, CPF/CNPJ com ou sem pontuação, telefone em vários formatos, CEP. Normalizar internamente e validar com mensagem clara. |
| **Jakob** | Seguir padrões que o usuário já conhece (navegação inferior, pull-to-refresh, mapas no padrão Google Maps/Waze, câmera nativa). Não reinventar controles. |
| **Frequência de uso** | Ações mais frequentes com menos toques e posição mais acessível; ações raras podem ir para menus. Use dados/analytics do app se existirem. |

## Parte 2: princípios de animação (com moderação)

Aplicar só onde comunica estado ou orienta o usuário. Em app operacional, animação é funcional, não decorativa.

- **Timing / slow in e slow out:** transições de 150 a 300 ms com easing (ease-out para entrada, ease-in para saída). Nada acima de 400 ms em fluxo operacional.
- **Feedback (exagerado só no essencial):** confirmação clara e marcante em eventos críticos: entrega confirmada, assinatura salva, ocorrência enviada (check animado + vibração háptica curta). Erro com leve shake e mensagem.
- **Anticipation / follow through:** transições de tela e bottom sheets que indicam de onde vêm e para onde vão.
- **Staging:** destacar a próxima ação (ex.: botão "Próxima entrega" ganha destaque quando a atual é concluída).
- **Secondary action:** pequenos sinais auxiliares, como badge pulsando para nova coleta atribuída, sem distrair.
- **Arcos:** útil em movimento de marcador no mapa/rota; não forçar em UI comum.
- **Squash and stretch, pose to pose, solid drawing, appeal:** baixa prioridade no TPKLOG. Use apenas em ilustrações de estado vazio, onboarding ou ícones, se o design system permitir.

## Regras obrigatórias

- Respeitar `prefers-reduced-motion` (web) e a configuração de reduzir movimento do sistema (iOS/Android). Com ela ativa, trocar animações por mudanças instantâneas ou fade curto.
- Animar só `transform` e `opacity` na web; usar thread nativa no mobile (ex.: Reanimated worklets, `useNativeDriver`). Nada que derrube FPS em aparelhos Android de entrada, comuns entre motoristas.
- Contraste e legibilidade sob sol: não depender de animação ou cor sozinha para comunicar estado.
- Nenhuma animação pode atrasar o envio de dados ou bloquear a próxima ação.

## Saída esperada

1. Relatório curto por tela: lei/princípio, problema encontrado (arquivo:linha), correção proposta, impacto (alto/médio/baixo).
2. Aplicar as correções de impacto alto e médio que não mudam fluxo de negócio; listar as demais para decisão.
3. Rodar lint, type check e testes existentes após as mudanças.
4. Se nada for pertinente em uma tela, dizer isso explicitamente em vez de forçar mudanças.
