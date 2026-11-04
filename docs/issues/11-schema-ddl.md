<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/12. A fonte oficial é o GitHub. -->

# 11: Schema de destino a partir da DDL

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

O usuário fornece a DDL da tabela de destino (arquivo ou colada) e a CLI mostra o que entendeu: colunas, tipos, tamanhos, NOT NULL, DEFAULT e auto-incremento.

## Acceptance criteria

- [ ] `schema` lê a DDL de `fatinstituicao` e `Medico` de exemplo e lista suas colunas
- [ ] Tipo, tamanho, NOT NULL, DEFAULT e auto-incremento são identificados
- [ ] Uma DDL com `IdTmp` é reconhecida como tal
- [ ] DDL inválida gera erro claro

## Blocked by

- [02](02-ponta-a-ponta-minimo.md) (#3) — Primeira migração ponta a ponta (mínima)
