# tpklog-skills

Skills do Claude Code para o desenvolvimento do **TPKLOG**, geradas na "fábrica de skill" a partir de estudos e vídeos.
Cada skill fica em `skills/<nome>/SKILL.md` e pode ser copiada direto para `.claude/skills/` do projeto TPKLOG.

## Skills

| Skill | Para que serve |
|---|---|
| `tpklog-padrao-base` | **Skill mestra**: padrão mínimo para todo projeto novo nascer no nível mais avançado, e registro dos aprendizados validados |
| `tpklog-performance` | Auditoria de performance em 5 camadas (tela, rede, servidor, banco, infra) + observabilidade |
| `tpklog-ux-motion` | Leis de UX e microinterações aplicadas às telas do TPKLOG |
| `tpklog-kit-ia` | Avaliar e adotar com segurança ferramentas de IA de terceiros no fluxo de desenvolvimento |
| `tpklog-harness-ia` | Ambiente de trabalho do Claude Code: AGENTS/CLAUDE enxutos, documentação com dono, portões de qualidade, revisão multiagente, ondas paralelas, memória/handoff e revisão de segurança |
| `tpklog-ui-visual` | Padrão visual dos painéis web (tokens, grid bento, KPIs, quadro de cargas por etapa, sidebar com badges, documento com selo de status); o que não adotar no app do motorista |
| `tpklog-baixa-whatsapp` | Baixa de canhoto por foto no WhatsApp: cascata de identificação (QR próprio, código de barras, OCR, contexto, escolha do motorista), confirmação, conferência, auditoria e travas de custo |
| `tpklog-autopilot` | Cria o comando `/autopilot`: todo pedido roda o mesmo fluxo de 10 etapas (brainstorm, plano, TDD, ciclos de review, PR, merge, limpeza) |
| `tpklog-roteirizador-ia` | Roteirizador automático: diretrizes por texto/voz viram regras (Claude), montador determinístico com rotas e corredores, rascunho no quadro, aprendizado com os ajustes do operador |

## Como instalar (no terminal, dentro do projeto TPKLOG)

```bash
# primeira vez
git clone https://github.com/marcelomartinsnader-hub/tpklog-skills.git ../tpklog-skills
# nas próximas vezes
git -C ../tpklog-skills pull

mkdir -p .claude/skills
cp -r ../tpklog-skills/skills/* .claude/skills/
```

## Como o Claude Code deve avaliar

As skills são **recomendações genéricas** até serem conferidas contra o código real. Para cada recomendação, o Claude Code classifica:

- **Já feito**: o código já atende (citar arquivo/linha).
- **Pertinente**: vale aplicar no TPKLOG (dizer onde e o esforço).
- **Não se aplica**: não faz sentido para a stack ou o produto (dizer por quê).
- **Conflita**: contradiz algo existente no projeto (dizer o quê).

Depois ajusta o texto da skill instalada ao projeto real e traz uma lista priorizada. **Nenhuma alteração no código do app antes da aprovação.**

## Ciclo de aprendizado

1. A fábrica gera ou atualiza uma skill temática e commita aqui.
2. O Claude Code puxa, avalia contra o código e aplica só o que for aprovado.
3. O que foi aprovado e aplicado volta para cá: o terminal atualiza `tpklog-padrao-base`, registra em `CHANGELOG.md` e faz commit/push.
4. Projeto novo começa pela `tpklog-padrao-base` e já nasce com tudo isso.

Tema já coberto → atualizar a skill existente em vez de criar outra. Nada de segredos ou dados de cliente aqui.
