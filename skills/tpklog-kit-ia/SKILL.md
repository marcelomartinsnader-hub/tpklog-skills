---
name: tpklog-kit-ia
description: Avaliar com segurança e, com aprovação, adotar no desenvolvimento do TPKLOG ferramentas de IA em alta no GitHub (superpowers, impeccable, mattpocock/skills, claude-mem). Use quando pedirem para melhorar o método de trabalho do Claude Code no TPKLOG, revisar o visual das telas ou instalar skills/plugins de terceiros.
---

# Kit de IA para desenvolver o TPKLOG: avaliar, pilotar e adotar

Origem: reel @inema.tds "Top 10 bombando no GitHub" (30/10/2026). Destes 10 repositórios, só 4 ajudam a **forma de desenvolver** o TPKLOG (nenhum é funcionalidade do app em si). Os demais foram descartados de propósito, veja o fim do arquivo.

| Repositório | O que promete | Para que serve no TPKLOG |
|---|---|---|
| `obra/superpowers` | Kit que faz a IA planejar, testar (TDD) e revisar antes de entregar | Método padrão para features novas (ex.: nova regra de frete, novo status de entrega) |
| `pbakaus/impeccable` | Skills de design para a IA caprichar no visual e fugir da "cara de robô" | Revisão visual de painel de cargas, rastreamento, telas do motorista |
| `mattpocock/skills` | Coleção de skills que um dev experiente usa no dia a dia | Fonte de boas práticas (principalmente TypeScript, se for a stack) |
| `thedotmack/claude-mem` | Memória entre sessões do Claude Code | Lembrar decisões do projeto entre conversas (avaliar com cuidado, ver Segurança) |

## Regras de trabalho

1. **Nada é instalado sem aprovação explícita do usuário**, item por item. Esta skill propõe; o usuário decide.
2. **Leia antes de instalar.** Para cada repositório, leia README, licença, instruções de instalação e todo arquivo executável (hooks, scripts `.sh`/`.js`/`.py`, servidores MCP). Código de terceiros roda com as suas permissões.
3. **Escopo de projeto, não global.** Quando aprovado, instale no `.claude/` do repositório TPKLOG (ou em branch própria), nunca em `~/.claude` sem o usuário pedir.
4. **Fixe a versão** (commit SHA ou tag) e registre de onde veio, para dar para auditar e reverter.
5. **Não substitua o que já existe.** Se o TPKLOG já tem `CLAUDE.md`, skills, hooks ou padrões de teste/commit, eles têm prioridade; o kit novo se encaixa neles.
6. Descreva números de estrelas e popularidade como sinal, não como garantia de qualidade.

## Fase 1 — Inventário (somente leitura)

1. Descubra a stack do TPKLOG (`package.json`, `composer.json`, `requirements.txt`, `*.csproj`, `docker-compose*`) e se há front-end web/mobile.
2. Liste o que já existe de configuração de IA: `CLAUDE.md`, `.claude/settings*.json`, `.claude/skills/`, `.claude/agents/`, hooks, `.mcp.json`, e skills globais em `~/.claude/skills/` (só listar).
3. Veja como o time trabalha hoje: há testes? cobertura? lint? PR com revisão? padrão de commit?

## Fase 2 — Avaliação de cada candidato

Para cada um dos 4 repositórios, clone em uma pasta temporária **fora** do repositório TPKLOG (ex.: `/tmp/kit-ia/<nome>`), no último commit, e preencha em `docs/ia/AVALIACAO-KIT.md`:

- **O que é de fato** (skills em Markdown? plugin? hooks? servidor MCP? binário?).
- **Licença** e se permite uso comercial.
- **Segurança**: lista de tudo que executa código, acessa rede, lê/grava fora do projeto, ou guarda dados. Para `claude-mem` em especial: onde a memória fica salva, se sai da máquina, se pode capturar segredos (`.env`, certificados digitais A1, senhas de SEFAZ, tokens de transportadoras) e se dá para excluir pastas.
- **Encaixe no TPKLOG**: sobreposição com o que já existe (Fase 1), dependência de stack (ex.: skills só de TypeScript/React), custo de tokens/contexto.
- **Veredito**: Adotar / Pilotar / Só copiar ideias / Descartar, com 1 a 2 linhas de motivo.

Critérios específicos:
- `superpowers`: compare o fluxo dele (planejar, escrever teste antes, implementar, revisar) com o processo atual. Se o TPKLOG não tem testes, o ganho vem junto com criar a base de testes; diga isso.
- `impeccable`: só faz sentido se houver interface (web, app do motorista, portal do cliente). Confirme quais telas existem.
- `mattpocock/skills`: escolha **apenas** as skills que casam com a stack; não traga a coleção inteira.
- `claude-mem`: compare com a memória nativa (`CLAUDE.md`, memória do Claude Code). Se a nativa resolve, o veredito padrão é Descartar.

**Pare aqui e mostre a tabela de vereditos ao usuário. Peça aprovação de quais pilotar.**

## Fase 3 — Piloto (somente os aprovados)

Crie uma branch `chore/kit-ia-piloto`. Para cada item aprovado:

1. Instale no escopo do projeto, na versão fixada; registre em `docs/ia/AVALIACAO-KIT.md` (origem, SHA, arquivos adicionados).
2. Rode **uma tarefa real e pequena** do TPKLOG com ele, por exemplo:
   - `superpowers`: implementar uma melhoria pequena e testável (ex.: validação de CEP/CNPJ no cadastro de destinatário, ou cálculo de prazo de entrega) seguindo plano → teste → código → revisão.
   - `impeccable`: auditar visualmente 1 tela de alto uso (painel/listagem de cargas ou tela de rastreamento) e propor ajustes; **aplicar só o que o usuário aprovar**.
   - skills do `mattpocock`: usar na revisão de um módulo existente.
3. Compare com o jeito atual: tempo, qualidade do resultado, testes criados, ruído/custo de contexto.
4. Testes e lint têm que continuar passando. Nunca desative teste para ficar verde.

## Fase 4 — Decisão e entrega

- Atualize `docs/ia/AVALIACAO-KIT.md` com o resultado do piloto e recomendação final (manter / remover).
- Se algo for mantido, adicione ao `CLAUDE.md` do TPKLOG 2 a 3 linhas dizendo quando usar.
- Remova da branch o que não passou no piloto.
- Entregue ao usuário um resumo curto: o que foi avaliado, o que ficou, o que saiu e por quê. Abra PR em vez de fazer merge direto.

## Descartados do reel (não avaliar sem pedido do usuário)

- `togg53192-cmd/jailbreaks`: textos para burlar travas de segurança de IAs. **Não usar**: risco ético, jurídico e de segurança, nada a ver com o TPKLOG.
- `ziwei-doushu`: astrologia chinesa, sem relação.
- `HKUDS/DeepTutor`: tutor de estudos com IA; útil só para treinamento pessoal, não para o app.
- `milind-soni/OpenMausBot`: "time de robôs" com computador próprio; agentes autônomos com acesso amplo, risco alto e sem caso de uso claro hoje.
- `pingdotgg/t3code`: controlar a IA pelo celular; conveniência pessoal, não muda o código do TPKLOG.
- `JetBrains/thinkrail`: IDE com IA; ferramenta alternativa, decisão de time, fora do repositório.
