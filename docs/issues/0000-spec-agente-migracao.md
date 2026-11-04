<!-- Espelho de https://github.com/William-SWS/data-migration-agent/issues/1. A fonte oficial é o GitHub. -->

# Spec: agente de migração de arquivos de origem para MySQL

## Problem Statement

O usuário recebe com frequência dados de clientes em arquivos CSV ou Excel e precisa levá-los para as tabelas do seu sistema em MySQL 8. Hoje esse trabalho é manual e repetitivo:

- Os arquivos de origem não são uniformes: uns separados por vírgula, outros por ponto e vírgula, em encodings diferentes, com datas em dd/mm/aa, decimais com vírgula e CPF/CNPJ com pontuação ou sem os zeros à esquerda que o Excel removeu.
- Os nomes das colunas da origem nunca batem com os das tabelas de destino, então cada migração exige montar à mão as equivalências, as tabelas de códigos (título "Dr." → 1, conselho "CRM" → 5, sexo...) e os valores de preenchimento das colunas que não existem na origem.
- O script de migração (`ALTER TABLE ... ADD COLUMN IdTmp` → `INSERT ... SELECT` → `DROP TABLE`) é escrito à mão a cada vez, e é fácil errar uma conversão de tipo, esquecer uma coluna ou trocar um código.
- Não há visibilidade prévia de quantos registros cada limpeza vai alterar ou remover.
- O que foi aprendido numa migração (que "Nome Fantasia" corresponde a `NomFantasia`, que o título vira código) se perde e precisa ser refeito na próxima.
- Os arquivos contêm dados pessoais (CPF, nomes), então não podem simplesmente ser entregues inteiros a um modelo de linguagem.

## Solution

Um agente formado por duas partes:

1. Uma **CLI Python determinística** que inspeciona o arquivo de origem, propõe um plano de limpeza com a contagem e amostras mascaradas de cada regra, aplica só as regras que o usuário aprovou, produz o CSV limpo (separado por `;`, UTF-8, datas `aaaa-mm-dd`, documentos só com dígitos, células vazias como NULL), lê o schema de destino a partir da DDL, sugere as equivalências de colunas e gera o script de migração a partir de um mapeamento YAML.
2. Uma **skill do Claude Code (`/migrar`)** que conduz esse fluxo conversando com o usuário, completa por semântica as equivalências que a CLI não resolveu e propõe acréscimos à memória de padrões, vendo apenas cabeçalhos, perfil das colunas e amostras mascaradas.

Entre a limpeza e o script, o usuário importa o CSV limpo no MySQL pelo DBeaver, criando a tabela externa com todas as colunas em `VARCHAR(500)`. O agente nunca executa nada no banco: o script de migração é entregue ao analista, que o executa em produção.

A cada migração, o que o usuário confirma (sinônimos de colunas, tabelas de códigos, valores de preenchimento, regras de separação) é gravado na memória de padrões, em YAML versionado, e reaproveitado nas migrações seguintes.

O projeto é construído como um curso: cada ticket é uma aula que o próprio usuário implementa.

## User Stories

### Leitura do arquivo de origem

1. Como usuário, quero passar um CSV separado por vírgula ou por ponto e vírgula sem dizer qual é, para não ter que abrir o arquivo antes.
2. Como usuário, quero que o encoding do arquivo de origem (UTF-8, Latin-1, Windows-1252) seja detectado, para que acentos não saiam corrompidos.
3. Como usuário, quero que o CSV limpo saia sempre em UTF-8, para que a importação no DBeaver seja previsível.
4. Como usuário, quero ler um `.xlsx` diretamente, escolhendo a aba, para não ter que exportá-lo para CSV antes.
5. Como usuário, quero que todas as colunas sejam lidas como texto, para que zeros à esquerda e códigos não sejam alterados na leitura.
6. Como usuário, quero processar arquivos de milhões de linhas sem esgotar a memória do computador, para usar o mesmo agente em qualquer tamanho de base.

### Inspeção e plano de limpeza

7. Como usuário, quero rodar uma inspeção que não altera o arquivo de origem, para saber o que será mudado antes de mudar.
8. Como usuário, quero ver, para cada regra de limpeza, quantos registros ela modifica ou deleta, para avaliar o impacto antes de aprovar.
9. Como usuário, quero ver amostras de antes/depois de cada regra, para conferir que a regra faz o que eu espero.
10. Como usuário, quero que as amostras tenham os dados pessoais mascarados, para cumprir a LGPD mesmo quando o relatório for lido por um modelo de linguagem.
11. Como usuário, quero receber um plano de limpeza em YAML com todas as regras desaprovadas por padrão, para aprovar uma a uma conscientemente.
12. Como usuário, quero aplicar só as regras aprovadas, para ter controle total do que acontece com os dados.
13. Como usuário, quero que nenhuma linha seja removida sem uma regra explicitamente aprovada, para nunca perder dados em silêncio.
14. Como usuário, quero que o relatório mostre linhas totalmente vazias sem removê-las automaticamente, para decidir o que fazer com elas.

### Regras de limpeza

15. Como usuário, quero que células vazias virem NULL no CSV limpo, sem afetar as outras células da linha, para que o banco receba NULL e não string vazia.
16. Como usuário, quero que datas de entrada sejam interpretadas como dd/mm e saiam como `aaaa-mm-dd`, para seguir o formato do meu banco.
17. Como usuário, quero que anos de 2 dígitos sejam resolvidos por uma regra de corte configurável, para não ter datas no século errado.
18. Como usuário, quero ver no relatório toda data inferida e toda data inválida, para revisar os casos duvidosos.
19. Como usuário, quero que decimais com vírgula (`1.234,56`) virem `1234.56`, para que o CAST no MySQL funcione.
20. Como usuário, quero que CPF e CNPJ saiam só com dígitos, para seguir o padrão do meu banco.
21. Como usuário, quero que CPF e CNPJ tenham os zeros à esquerda completados (11 e 14 dígitos), para recuperar o que o Excel removeu.
22. Como usuário, quero que os dígitos verificadores de CPF e CNPJ sejam validados, para saber quais documentos são inválidos.
23. Como usuário, quero decidir por regra o que fazer com os inválidos (manter, virar NULL ou excluir a linha), para não ter que decidir linha a linha em arquivos enormes.
24. Como usuário, quero que os inválidos sejam exportados para um CSV separado, para revisá-los ou corrigi-los manualmente.
25. Como usuário, quero que CEP, UF e telefone também sejam validados e normalizados, para aproveitar o mesmo cuidado em outros formatos conhecidos.
26. Como usuário, quero detectar linhas duplicadas idênticas e poder removê-las com aprovação, para não inserir registros repetidos.
27. Como usuário, quero ver no relatório linhas com o mesmo CPF/CNPJ sem que sejam removidas, porque podem ser registros diferentes.
28. Como usuário, quero que os cabeçalhos originais sejam mantidos como vieram, para que o mapeamento use os nomes que eu vejo no arquivo.
29. Como usuário, quero ser avisado de cabeçalhos que o MySQL não aceita (vazio, duplicado, mais de 64 caracteres) e poder aprovar a renomeação só deles, para que a importação no DBeaver não falhe.
30. Como usuário, quero ser avisado de valores com mais de 500 caracteres, para saber que não cabem na tabela externa.
31. Como usuário, quero ser avisado de valores maiores que o tamanho da coluna de destino, para evitar erro no modo estrito do MySQL.

### Regras de separação

32. Como usuário, quero extrair o título "Dr."/"Dra." do nome de um médico para uma coluna `titulo`, para que o título vire código no destino.
33. Como usuário, quero que as variações do título ("Dr", "DR.", "Dra", "Drª") sejam reconhecidas sem diferenciar maiúsculas, para não perder casos.
34. Como usuário, quero que um nome sem título deixe `titulo` vazio, para que ele vire NULL e depois o código 0.
35. Como usuário, quero que novas regras de separação possam ser aprendidas e guardadas conforme aparecem, para não reescrevê-las na próxima migração.

### Schema de destino

36. Como usuário, quero fornecer a DDL da tabela de destino (colada ou em arquivo), para que o agente conheça as colunas, os tipos, o NOT NULL e o auto-incremento.
37. Como usuário, quero, opcionalmente, que o agente leia o schema por uma conexão somente leitura, para não precisar exportar a DDL.
38. Como usuário, quero ter certeza de que o agente nunca executa ALTER, INSERT nem DROP no banco, para que nada mude sem passar pelo analista.

### Mapeamento e equivalências

39. Como usuário, quero receber um rascunho de mapeamento com as equivalências sugeridas, para não montar tudo do zero.
40. Como usuário, quero que as sugestões venham primeiro da memória de padrões e da similaridade de nomes, para que o resultado seja reproduzível.
41. Como usuário, quero que o Claude sugira as equivalências que sobraram, por significado, para cobrir nomes muito diferentes.
42. Como usuário, quero ver a origem de cada sugestão (memória, similaridade, Claude), para saber quanto confiar nela.
43. Como usuário, quero confirmar cada equivalência antes de gerar o script, para não migrar uma coluna para o lugar errado.
44. Como usuário, quero indicar qual coluna da origem é o ID de origem, para que ela alimente o `IdTmp`.
45. Como usuário, quero que o nome da tabela externa venha do nome do arquivo, podendo trocá-lo, para que o script aponte para a tabela que eu criei no DBeaver.

### Tabelas de códigos e valores de preenchimento

46. Como usuário, quero declarar que uma coluna de uma tabela de destino usa uma tabela de códigos e definir os pares valor → código, para configurar o agente uma vez.
47. Como usuário, quero poder declarar novas tabelas de códigos a qualquer momento, não só na configuração inicial, para acompanhar novos casos.
48. Como usuário, quero que valores sem código caiam em 0, para seguir o padrão atual dos meus scripts.
49. Como usuário, quero ver na inspeção quais valores cairiam no `ELSE 0` e poder incluí-los na tabela de códigos com confirmação, para não perder informação.
50. Como usuário, quero que colunas de destino sem equivalência recebam 0, ou uma data de preenchimento (por padrão `2000-01-01`) quando forem data, para que o INSERT não falhe.
51. Como usuário, quero configurar o valor de preenchimento por coluna e que ele seja lembrado, para não repetir a escolha.

### Script de migração

52. Como usuário, quero um único `.sql` por migração, cobrindo todas as tabelas de destino, para entregar um só arquivo ao analista.
53. Como usuário, quero que cada bloco siga o padrão `ALTER TABLE ... ADD COLUMN IdTmp INT` → `INSERT ... SELECT` → `DROP TABLE <externa>`, para manter o processo que já uso.
54. Como usuário, quero que o `ALTER` seja omitido, com um aviso comentado, quando a DDL já tiver `IdTmp`, para que o script não falhe numa segunda migração para a mesma tabela.
55. Como usuário, quero que toda coluna de origem entre como `NULLIF(TRIM(col), '')`, para que campos vazios virem NULL independentemente da configuração do DBeaver.
56. Como usuário, quero CAST explícito para o tipo de destino quando ele não for texto, para não depender da conversão implícita do MySQL.
57. Como usuário, quero que os códigos sejam aplicados por `CASE ... ELSE 0 END` no SQL, para que o analista veja a regra de forma legível.
58. Como usuário, quero que a chave primária auto-incremento fique fora do INSERT, para que o banco gere os IDs.
59. Como usuário, quero nomes de colunas sempre entre crases, para que cabeçalhos com espaço e acento funcionem.
60. Como analista, quero um cabeçalho comentado no script com arquivo de origem, data, número de linhas e regras aplicadas, para entender o que estou executando.
61. Como usuário, quero que os blocos sigam a ordem em que declarei os arquivos no mapeamento, para controlar a sequência de inserção.

### Memória de padrões

62. Como usuário, quero que sinônimos de colunas confirmados sejam guardados, para que a próxima migração sugira a mesma equivalência.
63. Como usuário, quero que a memória de padrões só mude com minha confirmação, para que nada seja aprendido errado em silêncio.
64. Como usuário, quero que a memória fique em arquivos YAML versionados e legíveis, para revisar no diff o que o agente aprendeu.
65. Como usuário, quero que a CLI valide os arquivos da memória, para que um erro de edição não quebre a próxima migração.

### Skill /migrar e privacidade

66. Como usuário, quero rodar `/migrar <arquivo>` e ser conduzido pelo fluxo inteiro (inspeção, aprovação, aplicação, importação, mapeamento, script, memória), para não precisar lembrar de cada comando.
67. Como usuário, quero que a skill só veja cabeçalhos, perfil das colunas e amostras mascaradas, nunca o arquivo bruto, para que dados pessoais não cheguem ao modelo.
68. Como usuário, quero que os dados de cada migração fiquem em `workspace/<migração>/`, ignorada pelo git, para que dados pessoais nunca sejam versionados.
69. Como usuário, quero que a CLI funcione sozinha, sem a skill, para poder usá-la mesmo sem o Claude Code.
70. Como aprendiz, quero que cada ticket seja uma aula curta que termina em algo que eu rodo e vejo funcionando, para aprender construindo o projeto.

## Implementation Decisions

### Arquitetura

- Núcleo Python determinístico, sem chamadas a LLM, exposto como CLI. Uma única skill do Claude Code orquestra a CLI. Sem uso da API do Claude.
- Os contratos entre a skill e o núcleo são arquivos: plano de limpeza (YAML), mapeamento (YAML), perfil das colunas (estruturado, legível por máquina) e memória de padrões (YAML).
- O agente nunca escreve no banco. O acesso ao banco é no máximo somente leitura (opcional, último ticket).

### Módulos

- **Leitor de origem**: recebe um caminho (e aba, para `.xlsx`) e devolve uma tabela preguiçosa (lazy) com todas as colunas como texto. Esconde a detecção de separador e encoding e a diferença entre CSV e Excel.
- **Formatos conhecidos**: para cada formato (data, decimal, CPF, CNPJ, CEP, UF, telefone), uma interface única "valor bruto → valor normalizado ou inválido (com motivo)". A regra de corte do ano de 2 dígitos é parâmetro.
- **Regras de limpeza e plano**: catálogo de regras (célula vazia → NULL, formatos conhecidos, separação de título, duplicatas, renomeação de cabeçalho inválido, tamanho de valores). `inspect` aplica cada regra em modo de contagem e produz o relatório de inspeção e o plano de limpeza com `approved: false`. `apply` lê o plano e executa só as regras aprovadas, gerando o CSV limpo e o CSV de inválidos. Inválidos têm ação por regra: manter, NULL ou excluir.
- **Mascaramento e perfil**: gera amostras mascaradas e o perfil das colunas (contagens, vazios, formatos detectados). É a única saída que a skill lê sobre os dados.
- **Schema de destino**: transforma DDL (via `sqlglot`) num modelo de tabela com colunas, tipo, tamanho, NOT NULL, DEFAULT e auto-incremento. Uma segunda fonte, opcional, lê o mesmo modelo do `information_schema` por conexão somente leitura.
- **Memória de padrões**: lê, valida e grava os YAML de sinônimos de colunas por tabela de destino, tabelas de códigos por coluna de destino, valores de preenchimento por coluna e regras de separação (incluindo as variações do título). Global, não por cliente. Toda gravação passa por confirmação.
- **Mapeamento**: modelo do YAML de mapeamento (por arquivo de origem: tabela externa, tabela de destino, ID de origem, e por coluna de destino: origem e transformação — direta, CAST, código, preenchimento). Inclui a sugestão determinística de equivalências (memória → nome normalizado sem acento/caixa → similaridade), marcando a origem de cada sugestão.
- **Gerador de script**: recebe mapeamento + schema de destino + memória de padrões e produz o script de migração. Aplica `NULLIF(TRIM())`, CAST por tipo, `CASE ... ELSE 0`, valores de preenchimento explícitos, exclusão do auto-incremento, crases, omissão do `ALTER` quando `IdTmp` já existe, um bloco por arquivo na ordem do mapeamento e cabeçalho comentado.
- **CLI**: comandos finos sobre os módulos acima (aproximadamente: `inspect`, `apply`, `profile`, `schema`, `suggest`, `sql`, `codes add`).
- **Skill `/migrar`**: instruções para conduzir o fluxo chamando a CLI, completar equivalências por semântica a partir do perfil e propor acréscimos à memória.

### Regras do script de migração

- Tabela externa é toda `VARCHAR(500)`, com os cabeçalhos originais.
- Valores de preenchimento: `0` para colunas que não são data; `'2000-01-01'` para datas, configurável por coluna. Sempre explícitos, mesmo com `DEFAULT` na DDL.
- Cada arquivo de origem alimenta exatamente uma tabela de destino.

### Stack

- Python 3.12+ com `uv`; `polars` em modo lazy/streaming; `fastexcel`/`openpyxl`; `charset-normalizer`; `sqlglot`; `PyYAML`; `Typer`; `pytest`.
- Código em inglês; termos do domínio vindos do banco em português; documentação em pt-BR.

## Testing Decisions

- Um bom teste exercita comportamento externo: dado um arquivo de origem (ou DDL, ou mapeamento) e um comando, verifica os arquivos produzidos (CSV limpo, relatório, plano, script). Não testa funções internas nem a forma como as regras são implementadas.
- **Costura principal: a CLI.** Os testes chamam os comandos da CLI em processo, sobre arquivos de fixture copiados para um diretório temporário, e comparam as saídas com o esperado. O script de migração é comparado com um arquivo esperado (golden file).
- **Costura secundária: formatos conhecidos.** A interface "valor bruto → normalizado ou inválido" é testada com tabelas de casos (CPF/CNPJ válidos e inválidos, datas com ano de 2 e 4 dígitos, decimais), porque a quantidade de casos tornaria os testes via CLI lentos e verbosos.
- Os fixtures são pequenos e fictícios (CPF/CNPJ gerados, nomes inventados). Nenhum dado real entra no repositório.
- Arquivos grandes: um teste gera um CSV sintético grande para verificar que inspeção e aplicação rodam em modo streaming sem materializar tudo.
- A skill `/migrar` não tem teste automatizado; é verificada manualmente, rodando o fluxo sobre os fixtures.
- Não há código ainda, então não há testes anteriores a seguir. O primeiro ticket estabelece o padrão.

## Out of Scope

- Outros SGBDs e migração de schema entre SGBDs.
- Executar qualquer comando de escrita no banco (ALTER, INSERT, DROP).
- Remapear chaves estrangeiras via `IdTmp` e remover a coluna `IdTmp` (`DROP COLUMN`).
- Gerar `CREATE TABLE` ou `LOAD DATA` para a tabela externa (a importação é feita no DBeaver).
- Arquivos `.xls` (formato antigo do Excel).
- Um arquivo de origem alimentando mais de uma tabela de destino.
- Chamadas à API do Claude e busca semântica/embeddings na memória de padrões.
- Tabelas de códigos pré-carregadas.

## Further Notes

- O projeto é um curso: cada ticket é uma aula escrita com a skill `mkdocs-project-course`, e o usuário implementa o código. Os tickets são pequenos (cerca de 20) e o primeiro após o ambiente é um fluxo ponta a ponta mínimo.
- Fonte das decisões: `docs/decisions.md` (entradas 1–61); vocabulário: `CONTEXT.md`.
- Ambiguidade menor registrada: no exemplo de `fatinstituicao` do brief, `DtaInclusao` recebeu `0`, o que contradiz a data de preenchimento padrão. Não bloqueia.
