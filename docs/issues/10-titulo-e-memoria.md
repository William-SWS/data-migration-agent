<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/11. A fonte oficial é o GitHub. -->

# 10: Separação do título e memória de padrões

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

A primeira regra de separação: o título Dr./Dra. sai do nome do médico e vai para a coluna `titulo` do CSV limpo, reconhecendo variações sem diferenciar maiúsculas. As variações ficam na memória de padrões, em YAML versionado, que nasce aqui com validação pela CLI. Nomes sem título deixam `titulo` vazio.

## Acceptance criteria

- [ ] `Dr. João Silva` vira nome `João Silva` e `titulo` `Dr.`
- [ ] Variações como `DR`, `dra`, `Drª` são reconhecidas
- [ ] As variações vêm da memória de padrões, não do código
- [ ] Um YAML de memória inválido gera erro claro na CLI
- [ ] Nome sem título deixa `titulo` vazio e conta no relatório

## Blocked by

- [04](04-inspecao-e-plano.md) (#5) — Inspeção e plano de limpeza
