<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/18. A fonte oficial é o GitHub. -->

# 17: Sugestão de equivalências

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

`suggest` monta o rascunho do mapeamento comparando os cabeçalhos do arquivo de origem com as colunas da tabela de destino: primeiro pelos sinônimos da memória, depois pelo nome normalizado (sem acento e caixa), depois por similaridade. Cada sugestão traz sua origem. Equivalências confirmadas viram sinônimos de coluna na memória.

## Acceptance criteria

- [ ] `Nome Fantasia` é sugerido para `NomFantasia` por similaridade
- [ ] Um sinônimo gravado na memória tem prioridade sobre a similaridade
- [ ] Cada equivalência no rascunho marca a origem (memória, nome, similaridade)
- [ ] Colunas sem sugestão ficam marcadas como pendentes
- [ ] Confirmar uma equivalência grava o sinônimo na memória

## Blocked by

- [10](10-titulo-e-memoria.md) (#11) — Separação do título e memória de padrões
- [12](12-script-tipado.md) (#13) — Script de migração com tipos
