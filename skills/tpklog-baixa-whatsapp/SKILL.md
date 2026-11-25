---
name: tpklog-baixa-whatsapp
description: Baixa de canhoto pelo WhatsApp no TPKLOG, como canal extra ao app do motorista - foto do canhoto, leitura da NF pelo Vision, confirmação do motorista, status com conferência e trilha de auditoria. Use ao avaliar, planejar ou implementar a baixa de entrega por WhatsApp, ou ao revisar se esse fluxo pode gravar status errado ou gerar custo no Google.
---

# Baixa de canhoto pelo WhatsApp

Origem: post de @laeltecnologia (25/11/2026, DgDtJi-AK0c). A dor: a equipe passa horas digitando baixa de canhoto, foto por foto. A ideia: o motorista manda a foto pelo WhatsApp, sem app e sem login. A IA lê a nota, quem recebeu, a assinatura e o carimbo, confere com as entregas do dia e devolve na hora a foto ruim.

No TPKLOG isso é **um canal a mais**, ao lado do app do motorista, nunca no lugar dele. As peças já existem no servidor (webhook e envio de WhatsApp, dispatcher de notificações, Vision lendo canhoto). A regra central: **a baixa pelo WhatsApp chama a mesma função de baixa do app**. Nada de segunda regra de negócio.

## Passo 0: reconhecer (somente leitura)

Antes de propor qualquer coisa, mapeie com arquivo e linha:
- Webhook do WhatsApp: qual provedor (Meta Cloud API ou intermediário pago)? Recebe imagem/mídia? Valida a assinatura do webhook? Tem idempotência pelo id da mensagem?
- Envio: dá para responder em texto livre dentro da janela de 24h, ou só por template?
- Vision hoje: qual função, qual recurso (`TEXT_DETECTION` ou `DOCUMENT_TEXT_DETECTION`), o que extrai e como casa com a NF.
- Baixa no app: qual função grava "Entregue", quais campos, se grava trilha (quem, quando, status anterior) e se existe desfazer.
- Cadastro do motorista: celular normalizado (E.164) e único, para identificar o remetente.
- Como listar as entregas em aberto de um motorista no dia (romaneio/viagem).
- Se existe alerta de orçamento no projeto GCP.
- Onde o TPKLOG gera romaneio/manifesto impresso (para o QR próprio) e se já há alguma biblioteca de leitura de QR/código de barras no servidor.
- Se o envio suporta mensagem interativa (botões/lista) do WhatsApp.

Classifique cada item como JÁ FEITO / PERTINENTE (onde, esforço P/M/G) / NÃO SE APLICA / CONFLITA. Não altere código sem aprovação. Itens aprovados entram na fila do `PLANO-SKILLS.md`, sem furar a fila.

## O fluxo (fase 1: nada é aprovado sozinho)

1. **Chega a foto.** O webhook responde 200 rápido e joga o trabalho para fila/job. Registra o id da mensagem: se já viu esse id, para aqui (a Meta reenvia).
2. **Quem mandou.** Número precisa ser de motorista cadastrado e ativo. Número desconhecido: resposta educada + fila manual no painel, sem OCR.
3. **Checagem barata antes da IA:** é imagem? resolução mínima? hash da imagem já usado em outra baixa? Foto repetida volta ao motorista sem chamar o Vision.
4. **Leitura pela cascata** (seção "Sempre achar o canhoto"): primeiro QR próprio e código de barras, de graça; o Vision (`DOCUMENT_TEXT_DETECTION`) só se precisar, uma única vez por imagem, guardando o texto bruto.
5. **Casamento restrito.** Procura a NF **só entre as entregas em aberto daquele motorista no dia**. Nenhuma ou mais de uma candidata: manda a lista dele para escolher, não adivinha.
6. **Foto ruim** (sem NF legível) volta na hora: "Não consegui ler a nota. Mande outra foto, com o canhoto inteiro, reto e com luz."
7. **Confirmação do motorista:** "NF 12345, Cliente X. Responda 1 para confirmar ou 2 se não for essa." Em seguida: "Teve ocorrência (avaria, falta, devolução)? 1 não, 2 sim." Sim leva para o fluxo de ocorrência, não para Entregue.
8. **Gravação pela mesma função do app**, com status **"Entregue - aguardando conferência"**, foto anexada e origem `whatsapp`.
9. **Conferência no painel:** operador vê foto + NF casada + texto lido e aprova (vira "Entregue") ou rejeita (volta para em aberto, com aviso ao motorista). Assinatura e carimbo são conferidos aqui: o Vision não valida assinatura de verdade.
10. **Fase 2 (só depois de medir):** auto-aprovar quando NF casada pela chave de 44 dígitos, única candidata, motorista confirmou e sem ocorrência. Só liberar com a taxa de rejeição da fase 1 em mãos.

## Sempre achar o canhoto: cascata de identificação

A concorrência promete "100% de canhoto baixado certo". O jeito honesto de chegar lá: **nenhuma foto fica sem dono** (sempre cai em uma NF ou em alguém que resolve) e **nenhuma baixa errada passa** (o que não tem certeza vai para confirmação). A identificação tenta, em ordem, da mais certa para a menos certa, e para no primeiro acerto:

1. **QR próprio do TPKLOG (o "algo a mais" que é nosso).** Ao emitir o romaneio/manifesto, o TPKLOG imprime um QR por entrega (etiqueta na via do DANFE ou no canhoto) com o id da entrega. Na foto, o QR é lido de graça no servidor (biblioteca de leitura de código, sem Vision). Acerto exato. Configurável por cliente: liga onde a operação aceita colar etiqueta.
2. **Código de barras da chave de acesso (44 dígitos)** do DANFE. Orientar o motorista a fotografar o canhoto junto com o topo do DANFE quando ainda estiverem presos. Leitura grátis, validar o dígito verificador. Acerto exato.
3. **OCR (Vision) da chave, ou do número + série da NF**, com correção dos erros comuns de leitura (O/0, I/1, S/5, B/8) e validação contra a lista de candidatas.
4. **Pontuação por contexto**, entre as entregas em aberto do motorista no dia: número da NF, série, CNPJ/nome do destinatário lidos, valor, data de emissão, ordem da rota (próxima parada) e horário. Só aceita se a melhor candidata tiver folga clara sobre a segunda.
5. **Escolha do motorista:** sem certeza, o WhatsApp manda a lista das NFs dele em aberto (mensagem interativa com botões/lista) e ele toca na certa. Esse passo garante que a foto sempre acha dono.
6. **Fila no painel:** o que sobrar (número desconhecido, motorista não respondeu em X minutos) aparece para o operador com a foto e as candidatas sugeridas.

### O que fica configurável (por cliente/embarcador)

- QR próprio ligado ou desligado, e onde imprimir.
- Modelo do canhoto: região onde ficam número, série, data e assinatura; padrão de número do emissor (regex), para ler melhor layouts diferentes.
- Campos exigidos para aceitar: assinatura, carimbo, nome legível, documento (RG/CPF), data do recebimento.
- Nível de confiança para auto-aprovar (fase 2) e quais clientes podem auto-aprovar.
- Tempo de espera pela resposta do motorista antes de ir para a fila do painel.
- Política de guarda da foto (prazo e classe de armazenamento).

### Assinatura e carimbo

O Vision lê texto, não confirma assinatura. Opção barata: checar se a região de assinatura do modelo configurado tem tinta (contagem de pixels escuros), e marcar "sem assinatura" para conferência. Modelo de IA multimodal para julgar assinatura/carimbo só se a conferência humana virar gargalo, medindo antes o custo por foto.

### Medir para poder prometer

Painel com, por dia e por cliente: % identificadas em cada degrau da cascata, % que precisou do motorista, % que foi para o painel, % rejeitadas na conferência, tempo médio da foto até a baixa. A promessa comercial sai desses números.

## Trilha de auditoria (obrigatória em toda baixa por WhatsApp)

id da mensagem, número remetente, motorista, hash e caminho da foto original, texto bruto do OCR, NF casada e como casou (chave, número+série, escolha do motorista), respostas do motorista com horário, status anterior e novo, quem conferiu e quando, e o desfazer pelo operador (que também gera registro). O WhatsApp remove o EXIF da foto, então não há GPS: se precisar de local, peça "compartilhar localização" como passo opcional.

## Riscos de gravar status errado e a trava de cada um

| Risco | Trava |
|---|---|
| OCR troca um dígito e casa com outra NF | casar só nas entregas do motorista no dia + confirmação dele + conferência |
| Foto reaproveitada ou de outro dia | hash da imagem; checar data quando legível |
| Webhook reenviado | idempotência pelo id da mensagem antes de qualquer OCR ou escrita |
| Número trocado, emprestado ou desconhecido | só números cadastrados e ativos; o resto vai para fila manual |
| Canhoto sem assinatura, com ressalva ou devolução parcial | pergunta de ocorrência + conferência humana; nunca auto-aprovar na fase 1 |
| Várias NFs numa foto ou várias fotos de uma vez | tratar cada imagem e cada NF separada, listar para confirmar |
| Corrida com baixa pelo app ao mesmo tempo | transação/condição "só baixa se ainda em aberto" na mesma função |

## Custo (para não pesar na conta do Google)

Referência de preço público (confira a tabela atual):
- Vision OCR: 1.000 imagens/mês grátis, depois cerca de US$ 1,50 por 1.000 (perto de R$ 0,01 por foto).
- WhatsApp Cloud API: conversa iniciada pelo motorista abre janela de 24h com respostas livres grátis. Template fora da janela é cobrado; intermediário pago cobra à parte.
- Storage: comprimir a foto (~200-400 KB) antes de guardar; regra de ciclo de vida para classe mais barata depois de 90 dias, se a política de guarda permitir.
- Firestore/Functions: poucas leituras e escritas por foto. Leia só as entregas do motorista no dia, nunca a coleção inteira.

O fluxo em si custa pouco. O que estoura a conta é bug. Travas obrigatórias:
- Idempotência antes do OCR, nunca depois.
- OCR só em imagem de número cadastrado que passou na checagem barata.
- Uma chamada ao Vision por imagem (guardar o resultado e reutilizar em reprocessamento).
- Limite por motorista por dia (ex.: 60 fotos) com alerta.
- Alerta de orçamento no GCP e um contador diário de chamadas ao Vision no painel.

## Como conferir que ficou certo

- Mandar a mesma foto duas vezes: segunda volta como repetida, uma só chamada ao Vision.
- Reenviar o mesmo payload do webhook: nada muda.
- Número não cadastrado: nenhuma chamada ao Vision, cai na fila manual.
- Foto de NF de outro motorista: não baixa.
- Responder 2 ("não é essa"): nada gravado, motorista recebe a lista.
- Baixa pelo app e pelo WhatsApp ao mesmo tempo: só uma vale, a outra recebe aviso.
- Desfazer no painel: status volta e a trilha registra quem desfez.
- Foto com QR próprio ou código de barras legível: baixa identificada sem chamar o Vision.
- Foto ilegível de motorista com 3 NFs em aberto: ele recebe a lista e escolhe; nenhuma foto fica sem dono.
