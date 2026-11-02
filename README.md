# data-migration-agent

Agente que ajuda a migrar dados recebidos em CSV ou Excel para o banco MySQL do sistema: limpa os arquivos e sugere as queries SQL de migração.

> Status: projeto em planejamento. Ainda não há código. Ele será construído como um curso, um ticket por aula.

## O problema

Bases de dados chegam em CSV, Excel etc. e precisam ir para o banco do sistema. Sempre aparecem os mesmos problemas:

- alguns arquivos são separados por vírgula e outros por ponto e vírgula, sem uniformidade;
- os nomes das colunas de cada arquivo não batem com os das colunas do banco, então é preciso montar uma equivalência, seja por nome fixo, seja por códigos.

A migração é feita com SQL: o arquivo, já limpo, é importado como uma tabela externa; os valores são selecionados com `SELECT` e inseridos nas tabelas do sistema com `INSERT`. A tabela de destino sempre recebe a coluna temporária `IdTmp`, e as colunas que existem no banco mas não na origem entram como `0`. O padrão, resumido a partir dos exemplos do brief (tabelas reais como `fatinstituicao` ← `procedencias` e `Medico` ← `profissionais`):

```sql
ALTER TABLE <tabela_de_destino> ADD COLUMN IdTmp INT;
INSERT INTO <tabela_de_destino> (<colunas_de_destino>, IdTmp)
SELECT <coluna_de_origem> AS <coluna_de_destino>,
CASE
 WHEN titulo = 'Dr.' THEN 1
 WHEN titulo = 'Dra.' THEN 2
 ELSE 0 END AS CodTitulo,
0 AS <coluna_sem_equivalente_na_origem>,
<id_de_origem> AS IdTmp
FROM <tabela_externa>;
DROP TABLE <tabela_externa>;
```

## Objetivo

Um agente cujo foco é ajudar na limpeza dos dados e sugerir as queries. Os dados limpos devem:

- estar separados por ponto e vírgula;
- ter datas no formato `aaaa-mm-dd`, sem nunca abreviar o ano;
- ter as células vazias como NULL;
- ter os formatos conhecidos verificados, principalmente CPF e CNPJ, os dois sem pontos ou traços.

E o agente deve:

- sempre informar quantos registros vão precisar ser modificados ou deletados;
- conseguir mover o título Dr./Dra. do nome de um médico para outra coluna e permitir criar outras regras de separação de textos;
- guardar e entender os padrões que já usou (por exemplo: o título normalmente vira código; o sexo também é migrado como código).

## Escopo

Só MySQL. Outros bancos de dados e migração de schema entre SGBDs não são prioridade por agora.

## Documentação

- [`CONTEXT.md`](CONTEXT.md): vocabulário do domínio.
- [`docs/decisions.md`](docs/decisions.md): log cronológico das decisões do projeto.
- [`AGENTS.md`](AGENTS.md): orientações para agentes de IA que trabalham neste repositório.
