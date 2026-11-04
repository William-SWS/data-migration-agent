<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/3. A fonte oficial é o GitHub. -->

# 02: Primeira migração ponta a ponta (mínima)

**Status:** aula

## Parent

[Spec](0000-spec-agente-migracao.md) (#1)

## What to build

Um fluxo completo, ainda sem validações: a CLI lê um arquivo de origem CSV separado por vírgula e grava o CSV limpo separado por `;`; depois, a partir de um mapeamento YAML escrito à mão (tabela externa, tabela de destino, ID de origem e equivalências diretas), gera o script de migração no padrão `ALTER TABLE ... ADD COLUMN IdTmp INT` → `INSERT ... SELECT` → `DROP TABLE <externa>`.

## Acceptance criteria

- [ ] Um CSV fictício com `,` vira um CSV limpo com `;`, mesmos cabeçalhos e mesmas linhas
- [ ] Um mapeamento YAML mínimo gera um `.sql` com os três comandos na ordem certa
- [ ] O ID de origem aparece como `... AS IdTmp` no SELECT
- [ ] Nomes de colunas aparecem entre crases
- [ ] Teste pela CLI compara o `.sql` gerado com um arquivo esperado

## Blocked by

- [01](01-ambiente.md) (#2) — Ambiente do projeto
