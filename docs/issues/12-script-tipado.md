<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/13. A fonte oficial é o GitHub. -->

# 12: Script de migração com tipos

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

O script de migração passa a usar o schema de destino: toda coluna de origem entra como `NULLIF(TRIM(col), '')`, com CAST explícito quando o destino não é texto; colunas de destino sem equivalência recebem o valor de preenchimento explícito (`0`, ou `'2000-01-01'` para datas, configurável por coluna); a chave auto-incremento fica fora do INSERT; se a DDL já tem `IdTmp`, o `ALTER` é omitido com um aviso comentado.

## Acceptance criteria

- [ ] Colunas DATE, DECIMAL e INT recebem o CAST correspondente
- [ ] Colunas sem equivalência recebem `0` ou a data de preenchimento
- [ ] O valor de preenchimento pode ser configurado por coluna
- [ ] A chave auto-incremento não aparece no INSERT
- [ ] Com `IdTmp` na DDL, o `ALTER` é omitido e há um comentário de aviso
- [ ] Script comparado com arquivo esperado

## Blocked by

- [11](11-schema-ddl.md) (#12) — Schema de destino a partir da DDL
