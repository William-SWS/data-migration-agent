<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/4. A fonte oficial é o GitHub. -->

# 03: Separador e encoding detectados

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

O mesmo fluxo aceita arquivos de origem separados por `,` ou `;` e em UTF-8, Latin-1 ou Windows-1252, sem o usuário informar nada. O CSV limpo sai sempre com `;` e em UTF-8, com acentos corretos.

## Acceptance criteria

- [ ] Arquivos com `,` e com `;` produzem o mesmo CSV limpo
- [ ] Um arquivo em Windows-1252 com acentos sai correto em UTF-8
- [ ] Todas as colunas são lidas como texto (zeros à esquerda preservados)
- [ ] A leitura é preguiçosa (lazy), preparada para arquivos grandes
- [ ] O separador e o encoding detectados aparecem na saída do comando

## Blocked by

- [02](02-ponta-a-ponta-minimo.md) (#3) — Primeira migração ponta a ponta (mínima)
