<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/19. A fonte oficial é o GitHub. -->

# 18: Skill /migrar

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

Uma skill do Claude Code, `/migrar <arquivo>`, conduz o fluxo inteiro chamando a CLI: inspeção, aprovação das regras com o usuário, aplicação, pausa para a importação no DBeaver, schema, sugestão de equivalências (completando por significado as pendentes), tabelas de códigos, script de migração e proposta de acréscimos à memória. Ela só lê o perfil e as amostras mascaradas.

## Acceptance criteria

- [ ] `/migrar` sobre os fixtures chega a um script de migração válido
- [ ] A skill nunca lê o arquivo de origem bruto nem o CSV limpo
- [ ] Equivalências sugeridas pelo Claude são marcadas como tal e confirmadas pelo usuário
- [ ] Acréscimos à memória só são gravados com confirmação
- [ ] Verificação manual documentada na aula

## Blocked by

- [05](05-amostras-e-perfil.md) (#6) — Amostras mascaradas e perfil das colunas
- [15](15-valores-sem-codigo.md) (#16) — Valores sem código
- [16](16-varios-arquivos.md) (#17) — Vários arquivos num único script
- [17](17-sugestao-de-equivalencias.md) (#18) — Sugestão de equivalências
