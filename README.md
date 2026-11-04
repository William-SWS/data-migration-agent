# data-migration-agent

Agente que ajuda a migrar dados recebidos em CSV ou Excel para o banco MySQL do sistema: limpa os arquivos e sugere as queries SQL de migração.

## O problema

Bases de dados chegam em CSV, Excel etc. e precisam ir para o banco do sistema. Sempre aparecem os mesmos problemas:

- alguns arquivos são separados por vírgula e outros por ponto e vírgula, sem uniformidade;
- os nomes das colunas de cada arquivo não batem com os das colunas do banco, então é preciso montar uma equivalência, seja por nome fixo, seja por códigos.

A migração é feita com SQL: o arquivo, já limpo, é importado como uma tabela externa; os valores são selecionados com `SELECT` e inseridos nas tabelas do sistema com `INSERT`. A tabela de destino sempre recebe a coluna temporária `IdTmp`, e as colunas que existem no banco mas não na origem entram como `0`. O padrão, ilustrado com um exemplo fictício de carros: a tabela de destino `Veiculo` recebe os dados da tabela externa `veiculos` (o CSV importado), cujas colunas têm outros nomes:

```sql
ALTER TABLE Veiculo ADD COLUMN IdTmp INT;
INSERT INTO Veiculo (NomMarca, NomModelo, CodCombustivel, AnoFabricacao, Placa, Inativo, IdTmp)
SELECT marca AS NomMarca,
modelo AS NomModelo,
CASE
 WHEN combustivel = 'Gasolina' THEN 1
 WHEN combustivel = 'Etanol' THEN 2
 WHEN combustivel = 'Flex' THEN 3
 ELSE 0 END AS CodCombustivel,
ano_fab AS AnoFabricacao,
placa AS Placa,
0 AS Inativo,
carro_id AS IdTmp
FROM veiculos;
DROP TABLE veiculos;
```

Aqui `combustivel` vira um código (como o título e o sexo no sistema real), `Inativo` não existe na origem e entra como `0`, e `carro_id` alimenta o `IdTmp`.

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