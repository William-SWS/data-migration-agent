<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/7. A fonte oficial é o GitHub. -->

# 06: Datas e decimais

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

Novas regras de limpeza: datas de entrada são interpretadas como dd/mm e saem como `aaaa-mm-dd`; anos com 2 dígitos usam uma regra de corte configurável; decimais com vírgula (`1.234,56`) viram `1234.56`. Datas inferidas e inválidas aparecem no relatório.

## Acceptance criteria

- [ ] `01/02/2020` vira `2020-02-01`
- [ ] `01/02/85` vira `1985-02-01` com o corte padrão, e o corte é configurável
- [ ] Datas impossíveis (ex.: `31/02/2020`) são contadas como inválidas, com amostra
- [ ] `1.234,56` vira `1234.56`
- [ ] Tabela de casos testa a interface do formato conhecido de data e de decimal

## Blocked by

- [04](04-inspecao-e-plano.md) (#5) — Inspeção e plano de limpeza
