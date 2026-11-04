<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/16. A fonte oficial é o GitHub. -->

# 15: Valores sem código

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

A inspeção conta os valores distintos que cairiam no `ELSE 0` de uma tabela de códigos e oferece incluí-los. Cada inclusão exige confirmação e vira memória de padrões.

## Acceptance criteria

- [ ] Valores sem código são listados com suas contagens
- [ ] Incluir um valor exige confirmação e grava na memória
- [ ] Depois da inclusão, o script gera o novo `WHEN`

## Blocked by

- [14](14-tabelas-de-codigos.md) (#15) — Tabelas de códigos
