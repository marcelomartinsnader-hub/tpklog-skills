Cole no Claude Code, dentro da pasta do projeto TPKLOG (troque <sua-conta> pelo seu usuário do GitHub):

---

Quero instalar e avaliar as skills da nossa fábrica de skills no TPKLOG.

1. Se a pasta ../tpklog-skills não existir, rode `git clone https://github.com/<sua-conta>/tpklog-skills.git ../tpklog-skills`. Se existir, rode `git -C ../tpklog-skills pull`.
2. Copie cada pasta de ../tpklog-skills/skills/ para .claude/skills/ deste projeto (crie a pasta se faltar). Se já houver uma versão instalada, mostre o que mudou antes de sobrescrever.
3. Para cada skill, leia todas as recomendações e confira contra o código real deste repositório. Classifique cada uma como:
   - JÁ FEITO (cite arquivo e linha que comprovam),
   - PERTINENTE (onde aplicar e esforço estimado: pequeno, médio ou grande),
   - NÃO SE APLICA (motivo, por exemplo stack diferente ou funcionalidade inexistente),
   - CONFLITA (com o quê no projeto e por quê).
4. Ajuste o texto das skills instaladas em .claude/skills/ ao que o TPKLOG realmente é: troque trechos genéricos pela stack, pastas, comandos e nomes reais do projeto, e remova o que não se aplica.
5. Me traga uma lista priorizada das recomendações PERTINENTES (impacto para o usuário e para o motorista em campo primeiro, depois esforço), com um parágrafo curto por item.

Importante: NÃO altere nenhum código do aplicativo agora. Só pode criar ou editar arquivos dentro de .claude/skills/. Espere eu aprovar os itens da lista antes de implementar qualquer coisa.
