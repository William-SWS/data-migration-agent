<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/8. A fonte oficial é o GitHub. -->

# 07: CPF, CNPJ, CEP, UF e telefone

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

Regras para os formatos conhecidos de documentos e contatos: CPF e CNPJ ficam só com dígitos, com zeros à esquerda completados (11 e 14) e dígitos verificadores validados; CEP, UF e telefone são normalizados e validados pelo mesmo mecanismo. O que fazer com os inválidos é decidido por regra no plano (manter, NULL ou excluir a linha), e os inválidos também são exportados para um CSV separado.

## Acceptance criteria

- [ ] `123.456.789-09` vira `12345678909`; um CPF com 10 dígitos ganha o zero à esquerda
- [ ] CPF/CNPJ com dígito verificador errado são contados como inválidos
- [ ] A ação para inválidos é escolhida no plano; excluir linhas só acontece se aprovada
- [ ] Os inválidos são exportados para um CSV separado em `workspace/<migração>/`
- [ ] CEP, UF e telefone são validados e normalizados
- [ ] Tabelas de casos cobrem válidos e inválidos de cada formato

## Blocked by

- [04](04-inspecao-e-plano.md) (#5) — Inspeção e plano de limpeza
