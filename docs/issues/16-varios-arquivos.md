<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/17. A fonte oficial é o GitHub. -->

# 16: Vários arquivos num único script

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

Uma migração pode ter vários arquivos de origem, cada um para uma tabela de destino. O script de migração é um único `.sql`, com um bloco por arquivo na ordem do mapeamento e um cabeçalho comentado para o analista (arquivos de origem, data, número de linhas, regras aplicadas).

## Acceptance criteria

- [ ] Um mapeamento com `procedencias` e `profissionais` gera um único `.sql` com dois blocos
- [ ] Os blocos seguem a ordem do mapeamento
- [ ] O cabeçalho comentado traz arquivos, data, linhas e regras aplicadas
- [ ] Script comparado com arquivo esperado

## Blocked by

- [12](12-script-tipado.md) (#13) — Script de migração com tipos
