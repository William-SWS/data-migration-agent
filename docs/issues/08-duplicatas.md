<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/9. A fonte oficial é o GitHub. -->

# 08: Duplicatas

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

A inspeção detecta linhas duplicadas idênticas, que podem ser removidas com aprovação, e linhas com o mesmo CPF/CNPJ, que são só relatadas porque podem ser registros diferentes.

## Acceptance criteria

- [ ] Duplicatas idênticas são contadas, com amostra mascarada
- [ ] A remoção de duplicatas idênticas só acontece com a regra aprovada
- [ ] Duplicatas por documento aparecem no relatório e nunca são removidas
- [ ] A comparação por documento usa o valor já normalizado

## Blocked by

- [07](07-documentos-e-contatos.md) (#8) — CPF, CNPJ, CEP, UF e telefone
