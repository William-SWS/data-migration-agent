<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/5. A fonte oficial é o GitHub. -->

# 04: Inspeção e plano de limpeza

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

A limpeza passa a ter duas etapas. `inspect` lê o arquivo de origem sem alterá-lo e produz o relatório de inspeção (quantos registros cada regra de limpeza altera) e o plano de limpeza em YAML, com todas as regras `approved: false`. `apply` executa só as regras aprovadas e grava o CSV limpo. A primeira regra é: célula vazia vira NULL (campo vazio), só aquela célula. Linhas totalmente vazias aparecem no relatório, mas não são removidas.

## Acceptance criteria

- [ ] `inspect` não altera o arquivo de origem e grava relatório + plano em `workspace/<migração>/`
- [ ] O relatório mostra, por regra, a contagem de registros afetados
- [ ] Com a regra desaprovada, `apply` não muda as células vazias; aprovada, muda
- [ ] Nenhuma linha é removida por esta regra
- [ ] Linhas totalmente vazias são contadas no relatório

## Blocked by

- [03](03-separador-e-encoding.md) (#4) — Separador e encoding detectados
