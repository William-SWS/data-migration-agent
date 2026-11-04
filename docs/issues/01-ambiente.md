<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/2. A fonte oficial é o GitHub. -->

# 01: Ambiente do projeto

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

O projeto passa a existir como pacote Python gerenciado pelo `uv`, com uma CLI `migra` instalada que responde a `--version`, e a suíte de testes roda com um primeiro teste verde. A pasta `workspace/` fica fora do git desde o início.

## Acceptance criteria

- [ ] `uv sync` cria o ambiente com as dependências de desenvolvimento
- [ ] `uv run migra --version` mostra a versão do projeto
- [ ] `uv run pytest` passa com um teste que chama a CLI em processo
- [ ] `workspace/` está no `.gitignore`
- [ ] `AGENTS.md` lista os comandos que passaram a funcionar

## Blocked by

- None (can start immediately)
