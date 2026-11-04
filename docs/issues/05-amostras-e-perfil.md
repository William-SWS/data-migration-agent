<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/6. A fonte oficial é o GitHub. -->

# 05: Amostras mascaradas e perfil das colunas

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

O relatório de inspeção passa a mostrar amostras de antes/depois de cada regra, com dados pessoais mascarados. Um comando `profile` produz o perfil das colunas (cabeçalhos, contagens, vazios, formatos detectados e amostras mascaradas) num formato legível por máquina: é a única visão dos dados que a skill vai ter.

## Acceptance criteria

- [ ] Cada regra no relatório traz amostras de antes/depois
- [ ] Nenhum valor de CPF, CNPJ, nome ou telefone aparece completo nas amostras
- [ ] `profile` gera o perfil de todas as colunas sem expor o arquivo bruto
- [ ] Teste garante que um valor sensível do fixture não aparece em nenhuma saída de `profile`

## Blocked by

- [04](04-inspecao-e-plano.md) (#5) — Inspeção e plano de limpeza
