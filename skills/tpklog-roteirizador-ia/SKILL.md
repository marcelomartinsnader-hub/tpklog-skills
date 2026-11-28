---
name: tpklog-roteirizador-ia
description: Roteirizador automático do TPKLOG com IA. Use ao planejar, construir ou revisar o botão "Sugerir carros", as diretrizes do operador por texto ou voz, o montador de carros a partir das tabelas de rotas e corredores, o rascunho no quadro e o aprendizado com os ajustes do operador.
---

# Roteirizador automático com IA (TPKLOG)

Objetivo do dono: apertar um botão, dar diretrizes do dia (texto ou áudio) e receber os carros montados como **rascunho**. O operador ajusta e aprova; o sistema aprende com os ajustes.

## Visão do dono: o app é o segundo cérebro do operador

Não é um botão que devolve resposta e não adivinha. É uma conversa contínua em que o sistema:
- **é proativo**: sugere sem ser pedido ("Muriaé tem só 8 NFs hoje, quer segurar até quarta?", "este carro passa da jornada, divido em 2?");
- **explica com dados do app** (médias por cidade, pernoites, rendimento das duplas, histórico, Maps) e cita a fonte de cada número;
- **debate alternativas e aceita contra-argumento**: refaz e mostra o custo da troca;
- **pergunta quando há dúvida**, nunca chuta;
- entrega carros como rascunho; o operador decide.

Interface: painel de conversa ao lado do rascunho dos carros (chat + quadro, como os líderes recomendam), com sugestões proativas que aparecem como cartões aceitar/recusar/discutir.

## Princípios (não negociáveis)

1. **A IA interpreta, o código monta.** O Claude só transforma a fala em regras estruturadas. Quem distribui NFs nos carros é um montador determinístico (mesma entrada → mesmo resultado), testável sem IA.
2. **Sempre rascunho.** O resultado vai para uma coleção de rascunho separada. Só o botão "Aplicar no quadro", clicado pelo operador, copia para o quadro oficial (gravando por linha).
3. **Confirmar antes de montar.** Toda diretriz interpretada aparece como "Entendi assim:" com cada regra editável. Ambiguidade → a IA pergunta, nunca chuta.
4. **Aprender só com aprovação.** Ajustes do operador e status ruins viram **sugestões** de regra. A regra só vale depois de aprovada. A IA nunca altera o montador nem cadastros sozinha.
5. **Nenhuma NF some.** O que não entrou em carro aparece em "sem carro" com o motivo (não coube, sem carro compatível, região segurada, motorista indisponível).
6. **O app é a fonte da verdade.** Cadastro dos carros (PERFIL: placa, capacidade, motorista, tipo de carga, regiões/categorias, fornecedores), tabela de rotas e corredor de rota. Nunca perguntar ao operador o que esses cadastros respondem. A planilha só mostra como a operação pensa.
7. **Usar o que já existe antes de criar dado novo:** histórico de rotas, entregas e status, endereços com coordenadas, janelas/dias de recebimento, tempos de parada e o Google Maps já integrado (distância, tempo, ordem de paradas).
8. **Conferir alinhamento antes de confiar:** carros × rotas × corredores × NFs. Montar carros só quando os itens graves zerarem.
9. **Chave da IA só no servidor** (Cloud Function + segredo do Firebase). Nada de chave no navegador nem no repositório.

## Como a operação roteiriza hoje (planilha, nov/2026)

- Abas por área: INTERIOR (agrupado por cidade, cidades vizinhas no mesmo carro, coluna ROTA vazia), JF MERCADOS (rotas "… MERCADOS", quase tudo Danone), JF VAREJO (rotas "JF · Centro", "JF · Zona Sul"…), TODAS NFS PARA CONTROLE.
- Colunas da NF: fornecedor, data inserida, OC, STATUS, NF, cliente, código do cliente, CNPJ, endereço, nº, bairro, cidade, ROTA, VOL (quase sempre vazio), PESO, VALOR, chave/pedido, telefone.
- Estrutura de um dia: linha do dia → cabeçalho do carro ("MOTORISTA / AJUDANTE OK") → NFs agrupadas por cliente → subtotal de peso. Carros saem perto do limite de kg.
- Fornecedores misturados no mesmo carro (DECMINAS, DVL, DANONE, BABY, VIGOR, ITA, SERRANA). Tipo de baú combina com a carga.
- Frota: tipo (Fiorino, Partner, Van, 3/4, Truck), baú (seca/resfriada/congelada), kg (550 a 8000). Escala: ESCALADO/LIVRE.
- Status que indicam erro de roteirização: NÃO COUBE, NÃO COUBE / ROTEIRIZADA, PASSOU PRA OUTRO CARRO EM ROTA, NÃO CARREGOU, ESQUECEU MERCADORIA ROTEIRIZADA, PERNOITE.
- O app já tem PERFIL por carro (placa, capacidade, motorista, categorias atendidas: Mercados JF / Varejo JF / Varejo interior / Mercados interior / todas; carga seca/refrig/congel; fornecedores). É a base de frota do roteirizador novo, mas as capacidades divergem entre telas e planilha (ex.: DAJ-8391 com 2.300, 3.500 e 2.500 kg): conferir antes de usar. "Carro DAJ" na fala = placa DAJ-8391.
- Problemas de dado a resolver no cadastro: nomes de motorista com grafias diferentes, CONGELADA/CONGELADO, cidades com caixa/acento variados, motoristas sem carro cadastrado.

## Regras fixas do cálculo (dono, 28/11)

- Objetivo: otimizar os carros (menos carros, bem cheios, dentro da jornada). A IA auxilia; o operador decide.
- **Mesmo dia:** utilitários (Fiorino, Partner, Montana, Saveiro, Kangoo, Doblo etc.) e todo carro com rota em Juiz de Fora saem e terminam no mesmo dia. Tempo da rota = direção pelo Google Maps + tempo de parada por loja, dentro da jornada.
- **Pernoite:** no interior, vans e caminhões (Van, HR, 3/4, Truck) podem ter rota de mais de um dia; o montador divide em dias, cada um dentro da jornada, e mostra o ponto de pernoite.
- **Números reais do app em vez de estimativa:** média de entregas por dia em cada cidade (limite de lojas por carro/cidade/dia), carros que mais pernoitam (base para decidir pernoite e qual carro mandar) e rendimento de ajudante × motorista (sugere a dupla; operador troca se quiser).
- Jornada (horário de saída, horas por dia, máximo de noites) vem do app se existir; senão é parâmetro configurado pelo dono.

## Conversa de esclarecimento (peça 0)

- O operador fala informal ("as vans do Heron", "o carro DAJ" = placa DAJ-8391, "a região de Muriaé", "os mercados"). O Claude cruza com o cadastro.
- Ambíguo → UMA pergunta curta por vez, com opções tocáveis ("Qual Carlos? Carlos Alberto (DAJ-8391) ou Carlos Eduardo (ATA7A84)?", "Muriaé inclui Miraí?"). Repete até não sobrar dúvida; aí mostra "Entendi assim".
- Respostas viram sinônimos salvos (ex.: "região de Muriaé" = lista de cidades) para não perguntar de novo; sinônimo também precisa de aprovação.
- Responde perguntas informais ("quem está livre hoje?", "o Sebastião leva refrigerado?") consultando o cadastro, sem inventar.
- Saída do Claude a cada turno: `{ "tipo": "pergunta", ... }` OU `{ "tipo": "regras", ... }`, validado por schema.

## Ideias dos líderes de mercado (pesquisa 28/11)

- Regras duras com motivo visível; nota órfã com motivo (Senior, Descartes).
- Rota fixa para cliente-chave (mesmo carro/motorista), dinâmica para o resto (Omnitracs).
- Preferência leve pelo motorista que conhece a rota (Routific, Wise).
- Aprender com edições e com sugestões aceitas/recusadas (Wise, Upper).
- Chat com pergunta de esclarecimento, simulação de impacto antes de aplicar, confirmação em lote, desfazer e registro de tudo (Upper, Route4Me).
- No rascunho: travar carro, arrastar loja, "encaixar no carro X", alerta ao estourar peso/carga/jornada (OptimoRoute, Routific).
- Placar por carro: peso e % de ocupação, nº de lojas, horas de rota, custo estimado, confiança (Senior).
- Explicar sobras e sugerir saída ("precisa de mais um 3/4") — HERE (protótipo).
- Previsto × realizado alimenta o aprendizado (Wise, OptimoRoute).

## Modelo de regras (saída do intérprete)

Cada diretriz vira um item com `tipo` fixo. Tipos mínimos:

| tipo | exemplo falado | campos |
|---|---|---|
| `carro_missao` | "começar pelas vans do Heron, Sebastião e Roberto com DECMINAS e DVL para as regiões mais longe" | motoristaIds[], tipoVeiculo?, fornecedores[], regioes[] ou `ordenarPor: "distancia_desc"`, prioridade |
| `regiao_fora` | "segunda não roteiriza Muriaé, esperar chegar mais" | data, regiaoId, motivo |
| `restricao_motorista` | "Cleiton, Emerson e Carlos só fazem JF na segunda" | motoristaIds[], data, areasPermitidas[] |
| `fornecedor_ancora` | "foco na Danone e põe junto os outros fornecedores das mesmas lojas" | fornecedorAncora, caronaMesmaLoja: true, escopo |
| `limite_lojas` | "JF nesses mercados, 5 lojas por carro" | escopo (área/rotas/motoristas), maxLojas |
| `pergunta` | qualquer ambiguidade | texto da pergunta, opções |

- Nomes falados são casados com os cadastros reais (lista de motoristas, fornecedores, rotas e regiões enviada no prompt). Sem casamento seguro → `pergunta`.
- "Loja" = endereço de entrega; várias NFs do mesmo cliente contam 1.
- Usar saída estruturada da API do Claude (`output_config.format` com JSON Schema, ou ferramenta com `strict: true`) e validar de novo no servidor. Modelo padrão `claude-opus-5-5`; tratar `stop_reason: "refusal"` e ligar fallback do lado do servidor.

## Montador determinístico (ordem dos passos)

1. NFs pendentes da data (status que ainda precisam sair).
2. Remover regiões `regiao_fora` → lista "segurada".
3. Agrupar NFs por loja (endereço de entrega). Loja nunca se divide entre carros.
4. Ligar cada loja a uma rota/corredor pelo cadastro existente (tabelas de rotas e corredores).
5. Carros disponíveis = escala do dia (ESCALADO/LIVRE) − restrições.
6. Atender `carro_missao` na ordem falada; depois os demais carros.
7. Encher cada carro seguindo a ordem do corredor até bater: kg (com folga configurável), jornada (tempo do Maps + parada; mesmo dia para utilitário e JF, pernoite permitido para van/caminhão no interior), `limite_lojas`, compatibilidade de baú × tipo de carga do fornecedor, área permitida do motorista.
8. `fornecedor_ancora`: toda loja da âncora no carro puxa as NFs de outros fornecedores da mesma loja, se o baú permitir.
9. Sobras → "sem carro" com motivo.
10. Sugerir o ajudante de cada carro pelo histórico de rendimento com aquele motorista (sugestão, não regra).
11. Gravar rascunho com, por carro: peso total, % da capacidade, nº de lojas e o motivo de cada escolha.

Testes obrigatórios: casos com os 5 exemplos acima; loja não dividida; nunca passar da capacidade; NF nunca perdida (entrada = carros + sem carro + segurada).

## Aprendizado

- Registrar cada montagem (diretrizes, regras confirmadas, rascunho) e o que foi aprovado.
- Gravar a diferença rascunho × aprovado como eventos: `moveu_loja`, `trocou_motorista`, `segurou_regiao`, `removeu_nf`, `adicionou_nf`.
- Juntar com os status ruins de entrega.
- Rotina periódica: a IA lê o histórico e propõe regras ("Sebastião não vai para X", "Bahamas CD sai sozinho") com a evidência (quantas vezes, quais dias). Operador aprova/recusa. Regras aprovadas entram no montador como restrições normais.

## Voz

- O Claude recebe texto: o áudio precisa ser transcrito antes.
- Microfone no sistema (reconhecimento do navegador ou serviço de transcrição) e áudio no bot do WhatsApp (mesmo canal da `tpklog-baixa-whatsapp`).
- O texto transcrito aparece para conferir antes de virar regra.

## Tentativas antigas (começar do zero)

O app tem duas tentativas: "Roteirização estilo planilha" (Simular roteirização automática, Preferências aprendidas, mapa, até 10% acima da capacidade; ignora tipo de carga/região e lê kg do nome do modelo) e "Roteirizar automático — carros disponíveis hoje" (no quadro; já respeitava tipo de carga, categoria, região, fornecedor e corredor, segundo o raio-X de 28/11). O dono decidiu **não aproveitar** e tirar as duas do app.
- Antes de remover: listar arquivos, funções, coleções do Firestore, menus/botões e dependências de cada uma, e o que vai embora.
- Proposta (decisão do dono): remover só telas e lógica; **manter os dados de cadastro** (PERFIL do carro, capacidade, vínculo carro-motorista). Preferências aprendidas antigas: apagar ou arquivar, conforme o dono decidir.
- Remoção vai pelo canal de teste; produção só com ok do dono.
- Não reaproveitar a lógica de SUGESTÃO delas. Peças de uso geral que moram perto e outras telas usam ficam (renomeadas se preciso): régua do tipo de carga pela NF/produto, régua de região/categoria, geocodificação e ordem de paradas do Maps, vehicleCapacityKg.
- Rota fixa por cliente: ideia dos concorrentes, mas o TPKLOG ainda não tem; fica para depois.

## Vocação do carro (frota terceirizada, muda muito)

Não há rota fixa, mas há carros **destinados ao mesmo tipo de trabalho**. Ex.: Emerson (GKR-5015) carrega preferencialmente Danone em mercados de JF; as lojas mudam porque o fornecedor não tem constância. Modelar em duas camadas do PERFIL:
- **Permitido (regra dura):** categorias (`perfilCategorias`: só `mercado_jf` → não viaja e não faz varejo JF), tipo de carga, fornecedores permitidos. O montador nunca viola.
- **Preferência (regra leve):** fornecedor preferencial (ex.: Danone), que o montador atende primeiro com esse carro e só usa para outro fornecedor quando sobrar espaço. O segundo cérebro avisa quando quebra a preferência ("Danone veio pouco hoje; completo o Emerson com mercados da Vigor?").
- **Regiões preferenciais em ordem, com plano B** (regra leve): ex.: Sebastião (RVB-7I28) → 1º região de Ervália; se lá não tiver carga, tiver pouco peso, ou houver prioridade em outro lugar → 2º Barbacena. O montador tenta a 1ª, cai para a 2ª pelos gatilhos (sem carga, peso abaixo do mínimo configurado, prioridade) e o segundo cérebro diz o motivo ("Ervália tem só 180 kg hoje; mando o Sebastião para Barbacena?").
- As vocações também podem nascer do histórico (onde cada carro mais rodou) como sugestão para o operador aprovar.
- **Meta do dono: sair do muito flexível para mais constância.** Indicador semanal de constância: % dos dias em que cada carro rodou na sua vocação (região/fornecedor 1ª opção, 2ª opção, fora). O segundo cérebro aponta quem está fugindo da vocação e sugere ajustar a vocação ou a montagem. Constância cresce aos poucos; nada vira fixo sem o dono aprovar.
- Nunca deduzir lojas fixas desse carro. A vocação é tipo de trabalho, não endereço.

## Fases

0. Raio-X do código + inventário do que o app já tem (só leitura). 0b. Remover as duas tentativas antigas (inventário primeiro, canal de teste, ok do dono). 1a. Conferência de alinhamento (carros × rotas × corredores × NFs). 1. Cadastros com ID único + registro de montagens e ajustes. 2. Botão "Sugerir carros" com diretrizes por formulário (sem IA). 3. Diretrizes por texto livre (Claude preenche o formulário). 4. Voz. 5. Aprendizado com aprovação. 6. Ordem de entrega dentro do carro (precisa geocodificar clientes).

Cada fase entra na fila do PLANO-SKILLS.md e só começa com aprovação do dono.

## Como o Claude Code deve avaliar esta skill

Para cada peça (cadastros, rotas/corredores, escala, quadro, rascunho, registro, intérprete, montador, voz, aprendizado) classificar: **Já feito** (arquivo:linha) / **Pertinente** (onde e esforço P/M/G) / **Não se aplica** (motivo) / **Conflita** (com o quê). Nenhuma alteração no código do app antes da aprovação.
