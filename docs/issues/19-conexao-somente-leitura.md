<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/20. A fonte oficial é o GitHub. -->

# 19: (Opcional) Schema pela conexão somente leitura

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

Alternativa à DDL: a CLI lê o schema de destino do `information_schema` por uma conexão MySQL somente leitura, produzindo o mesmo modelo de schema.

## Acceptance criteria

- [ ] Com credenciais de leitura, `schema` lista as mesmas informações que a DDL
- [ ] A CLI não oferece nenhum comando que escreva no banco
- [ ] Credenciais não são gravadas em arquivos versionados

## Blocked by

- [11](11-schema-ddl.md) (#12) — Schema de destino a partir da DDL
