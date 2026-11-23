# tpklog-skills

Skills do Claude Code para o desenvolvimento do **TPKLOG**, geradas na "fábrica de skill" a partir de estudos e vídeos.
Cada skill fica em `skills/<nome>/SKILL.md` e pode ser copiada direto para `.claude/skills/` do projeto TPKLOG.

## Skills

| Skill | Para que serve |
|---|---|
| `tpklog-performance` | Auditoria de performance em 5 camadas (tela, rede, servidor, banco, infra) + observabilidade |
| `tpklog-ux-motion` | Leis de UX e microinterações aplicadas às telas do TPKLOG |
| `tpklog-kit-ia` | Avaliar e adotar com segurança ferramentas de IA de terceiros no fluxo de desenvolvimento |

## Como instalar (no terminal, dentro do projeto TPKLOG)

```bash
# primeira vez
git clone https://github.com/<sua-conta>/tpklog-skills.git ../tpklog-skills
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

## Regra do repositório

Toda skill nova ou atualizada da fábrica é commitada aqui. Tema já coberto → atualizar a skill existente em vez de criar outra.
