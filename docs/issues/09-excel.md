<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/10. A fonte oficial é o GitHub. -->

# 09: Arquivos Excel (.xlsx)

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

O arquivo de origem pode ser um `.xlsx`. O usuário escolhe a aba (ou a primeira é usada) e o restante do fluxo é igual ao do CSV.

## Acceptance criteria

- [ ] Um `.xlsx` fictício produz o mesmo CSV limpo que o CSV equivalente
- [ ] É possível escolher a aba; sem escolha, usa a primeira e diz qual foi
- [ ] Abas disponíveis são listadas quando a escolhida não existe
- [ ] Todas as colunas são lidas como texto

## Blocked by

- [03](03-separador-e-encoding.md) (#4) — Separador e encoding detectados
