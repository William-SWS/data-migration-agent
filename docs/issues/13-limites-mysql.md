<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/14. A fonte oficial é o GitHub. -->

# 13: Inspeção contra os limites do MySQL

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

A inspeção sinaliza cabeçalhos que o MySQL não aceita (vazio, duplicado, mais de 64 caracteres) e propõe renomear só esses, com aprovação; e relata valores com mais de 500 caracteres e valores maiores que o tamanho da coluna de destino.

## Acceptance criteria

- [ ] Cabeçalhos inválidos geram uma regra de renomeação no plano; os demais ficam intactos
- [ ] A renomeação só acontece com a regra aprovada
- [ ] Valores com mais de 500 caracteres são contados, com amostra mascarada
- [ ] Com a DDL informada, valores maiores que a coluna de destino são contados

## Blocked by

- [04](04-inspecao-e-plano.md) (#5) — Inspeção e plano de limpeza
- [11](11-schema-ddl.md) (#12) — Schema de destino a partir da DDL
