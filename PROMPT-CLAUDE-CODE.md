# Prompts para o Claude Code


## A) Projeto existente (TPKLOG): puxar, avaliar e devolver o aprendizado

Cole no Claude Code, dentro da pasta do projeto TPKLOG:

---

Quero instalar e avaliar as skills da nossa fábrica de skills neste projeto.

1. Se a pasta ../tpklog-skills não existir, rode `git clone https://github.com/marcelomartinsnader-hub/tpklog-skills.git ../tpklog-skills`. Se existir, rode `git -C ../tpklog-skills pull`.
2. Copie cada pasta de ../tpklog-skills/skills/ para .claude/skills/ deste projeto (crie a pasta se faltar). Se já houver uma versão instalada, mostre o que mudou antes de sobrescrever.
3. Comece pela skill tpklog-padrao-base (modo "projeto existente") e depois pelas skills temáticas. Para cada recomendação, confira contra o código real e classifique como:
   - JÁ FEITO (cite arquivo e linha que comprovam),
   - PERTINENTE (onde aplicar e esforço: pequeno, médio ou grande),
   - NÃO SE APLICA (motivo),
   - CONFLITA (com o quê no projeto e por quê).
4. Ajuste o texto das skills instaladas em .claude/skills/ ao que este projeto realmente é (stack, pastas, comandos e nomes reais) e remova o que não se aplica.
5. Me traga uma lista priorizada das recomendações PERTINENTES (impacto para o usuário e para o motorista em campo primeiro, depois esforço), com um parágrafo curto por item.

Importante: NÃO altere nenhum código do aplicativo agora. Só pode criar ou editar arquivos dentro de .claude/skills/. Espere eu aprovar os itens antes de implementar.

Depois que eu aprovar e você aplicar um item, siga o "Ciclo de atualização" da skill tpklog-padrao-base: atualize ../tpklog-skills/skills/tpklog-padrao-base/SKILL.md (padrão mínimo e "Aprendizados validados") e o CHANGELOG.md, me mostre o diff e, com meu ok, faça commit e push no repositório tpklog-skills. Se uma recomendação se mostrou errada na prática, corrija também a skill temática. Nunca leve para lá segredos, dados de clientes ou código específico do TPKLOG, só o padrão reutilizável.

---

## B) Projeto novo: nascer no padrão

Cole no Claude Code, dentro da pasta vazia do projeto novo:

---

Vamos começar um projeto novo no padrão Transportech.

1. Rode `git clone https://github.com/marcelomartinsnader-hub/tpklog-skills.git ../tpklog-skills` (ou `git -C ../tpklog-skills pull` se já existir) e copie ../tpklog-skills/skills/* para .claude/skills/.
2. Siga a skill tpklog-padrao-base no modo "projeto novo": me faça as perguntas iniciais, proponha a estrutura com o padrão mínimo adaptado à stack e espere meu ok antes de criar os arquivos.
3. No fim, me diga o que já nasceu no padrão e o que ficou pendente, e registre no tpklog-skills qualquer aprendizado novo, seguindo o ciclo de atualização da skill.

---
