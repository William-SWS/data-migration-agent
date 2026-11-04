<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/15. A fonte oficial é o GitHub. -->

# 14: Tabelas de códigos

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

O usuário declara que uma coluna de uma tabela de destino usa uma tabela de códigos (`codes add <Tabela>.<Coluna>`) e informa os pares valor → código, que são gravados na memória de padrões. O script gera `CASE ... ELSE 0 END` para essa coluna. O comando pode ser usado a qualquer momento.

## Acceptance criteria

- [ ] `codes add Medico.CodTitulo` grava `Dr.` → 1 e `Dra.` → 2 na memória
- [ ] O script gera o CASE com esses pares e `ELSE 0`
- [ ] Um segundo `codes add` na mesma coluna acrescenta pares sem perder os anteriores
- [ ] Nada é gravado sem confirmação

## Blocked by

- [10](10-titulo-e-memoria.md) (#11) — Separação do título e memória de padrões
- [12](12-script-tipado.md) (#13) — Script de migração com tipos
