# Log de decisões

Registro cronológico e append-only das decisões do projeto. Uma seção `## AAAA-MM-DD` por dia; cada decisão nova entra no fim da seção do dia. Para revisar uma decisão anterior, adicione uma entrada nova que referencie a antiga pelo número, sem reescrever a original.

Complementa (não substitui) os ADRs em `docs/adr/` e as issues do tracker do projeto (ver `AGENTS.md`).

## 2026-11-01

Decisões extraídas do brief inicial do usuário (antes da entrevista):

1. **Escopo de banco: só MySQL.** Outros SGBDs e migração de schema entre SGBDs ficam fora de escopo por enquanto (não prioritários).
2. **Objetivo do agente:** auxiliar na limpeza dos dados de origem (CSV, Excel) e sugerir as queries SQL de migração no padrão `ALTER TABLE <destino> ADD COLUMN IdTmp INT;` → `INSERT INTO <destino> (...) SELECT ... FROM <tabela_externa>;` → `DROP TABLE <tabela_externa>;`.
3. **Padrão de migração:** a tabela de destino sempre recebe a coluna temporária `IdTmp`, preenchida com o ID original da base externa; colunas do destino que não existem na origem entram como `0` (conforme os exemplos do brief; a regra exata de valores padrão ainda será confirmada na entrevista).
4. **CSV limpo usa `;` como separador.**
5. **Datas no CSV limpo no formato `aaaa-mm-dd`**, nunca com o ano abreviado.
6. **CPF e CNPJ sem pontos, barras ou traços** no CSV limpo, e sempre verificados.
7. **Antes de alterar dados, o agente informa quantos registros serão modificados ou deletados.**
8. **Regras de separação de strings:** o agente deve conseguir mover o título (Dr./Dra.) do nome de um médico para outra coluna, e permitir criar outras regras de separação de strings.
9. **Memória de padrões (desejável):** o agente deve guardar e reaproveitar padrões de migrações anteriores (ex.: título e sexo costumam virar códigos).

Decisões da rodada 1 da entrevista:

10. **Arquitetura: núcleo Python determinístico + skill do Claude Code.** O núcleo Python faz limpeza, validação, contagem e geração de SQL a partir de um arquivo de mapeamento, sem LLM. Uma skill do Claude Code, por cima, sugere mapeamentos de colunas e lembra padrões. Sem uso da API do Claude (sem chave e sem custo por uso).
11. **O projeto é um curso.** O usuário implementa o código ele mesmo, aprendendo com uma aula (MkDocs) por ticket, escrita com a skill `mkdocs-project-course`. Implementar um ticket significa escrever a página do curso (`docs/NN-*.md`) com o código mostrado como conteúdo do tutorial, e nunca os arquivos reais em `src/` ou `tests/`. Código real só é escrito ou validado quando o usuário pedir explicitamente. Os tickets precisam ter tamanho e ordem de aula.
12. **Células vazias viram NULL no CSV limpo.** Essa é a única transformação de vazio. Nenhuma linha é removida automaticamente, nem as totalmente vazias; no máximo elas aparecem no relatório.
13. **Exemplos do brief valem como estão.** Os trechos que pareciam truncados são tratados como o padrão real do usuário; não bloqueiam o projeto.
14. **Escopo do IdTmp: só criar.** O script gerado segue `ALTER TABLE <destino> ADD COLUMN IdTmp INT;` → `INSERT ... SELECT ... FROM <externa>;` → `DROP TABLE <externa>;` e é entregue ao analista que o executa em produção. Remapear FKs via IdTmp e remover a coluna `IdTmp` (`DROP COLUMN`) ficam fora de escopo por enquanto.
15. **A importação do CSV limpo para o MySQL é feita pelo usuário no DBeaver.** O agente não gera `CREATE TABLE` nem `LOAD DATA` para a tabela externa.
16. **Acesso ao banco: no máximo somente leitura.** O agente também aceita a DDL do banco/tabela de destino (colada ou em arquivo) e entende o schema de destino a partir dela. Nunca executa ALTER, INSERT nem DROP: só gera o `.sql`.
17. **Excel na v1:** `.xlsx` lido direto, com escolha da aba. `.xls` fica para depois.
18. **Encoding e números:** o encoding de entrada é detectado e a saída é sempre UTF-8. Vírgula decimal é normalizada para ponto.
19. **CPF/CNPJ e outros formatos:** remover pontuação, completar zeros à esquerda (11/14 dígitos) e validar os dígitos verificadores. Inválidos vão para o relatório e são decididos caso a caso; nada é excluído automaticamente. A v1 também valida CEP, UF e telefone.
20. **Formato de data de saída fixo em `aaaa-mm-dd`, sem exceção** (reforça a entrada 5). Como interpretar datas de entrada ambíguas (dd/mm vs mm/dd, ano com 2 dígitos) ainda está em aberto.
21. **Fluxo de aprovação:** dry-run obrigatório com contagem e amostras por regra, aprovação regra a regra, e nenhuma deleção sem aprovação explícita (detalha a entrada 7).
22. **Colunas de destino ausentes na origem recebem `0` quando não são data** (esclarece a entrada 3). O valor padrão para colunas de data ainda está em aberto; o exemplo do brief usou `'2000-01-01'`.
23. **A tabela externa é sempre toda `VARCHAR(500)`.** Ao importar o CSV pelo DBeaver, o usuário define todas as colunas como `VARCHAR(500)`. Os `SELECT` gerados precisam considerar isso (conversões/casts para o tipo de destino).
24. **Separação de strings na v1: só a extração do título Dr./Dra.** (detalha a entrada 8). O agente precisa aprender regras novas conforme forem surgindo no uso, então a memória de padrões tem que ser extensível pelo uso.
25. **Contexto técnico:** MySQL 8.x. Arquivos de centenas a centenas de milhares de linhas, possivelmente milhões; o design deve suportar arquivos grandes (streaming/processamento em lotes).
26. **Spec e tickets publicados como arquivos locais em `docs/issues/`** (sem issue tracker).
27. **Idioma:** documentação em pt-BR, código em inglês, termos de domínio vindos do banco (`fatinstituicao`, `IdTmp`, `NomFantasia`...) mantidos em português.

## 2026-11-02

Decisões da rodada 2 da entrevista:

28. **Datas de entrada sempre no formato brasileiro (dd/mm).** Ano com 2 dígitos é resolvido por uma regra de corte configurável. Toda data inferida ou inválida aparece no dry-run (complementa a entrada 20).
29. **Colunas de data ausentes na origem recebem uma data de preenchimento**, por padrão `'2000-01-01'`, configurável por coluna (resolve a pendência da entrada 22). Motivo dado pelo usuário: `DtaCadastro` não existia na origem, então uma data arbitrária foi colocada no destino. Ambiguidade menor, que não bloqueia: no exemplo de `fatinstituicao`, `DtaInclusao` recebeu `0`, o que não foi explicado.
30. **NULL no CSV limpo é campo vazio.** O SQL gerado envolve cada coluna da origem em `NULLIF(TRIM(col), '')`, o que funciona com qualquer configuração de importação do DBeaver (detalha a entrada 12).
31. **CAST explícito no SELECT** para o tipo de destino (lido da DDL) sempre que ele não for texto, já que a tabela externa é toda `VARCHAR(500)` (entrada 23).
32. **Códigos aplicados via `CASE` no SQL**, como nos exemplos do brief. O CSV limpo guarda o valor textual limpo. `ELSE 0` é o padrão.
33. **Extração do título Dr./Dra.:** a limpeza cria a coluna `titulo` no CSV, remove o prefixo do nome e reconhece as variações sem diferenciar maiúsculas ("Dr", "DR.", "Dra", "Drª"...). Nome sem título deixa `titulo` vazio (→ NULL → `ELSE 0`).
34. **Tabelas de código não são semeadas agora.** Requisito: ao configurar/inicializar o agente, o usuário precisa conseguir declarar que uma coluna de uma tabela de destino recebe tratamento de tabela de código, e então definir os códigos dela.
35. **Memória de padrões em YAML versionado, em `patterns/`**: sinônimos de colunas por tabela de destino, tabelas de código, valores padrão por coluna e regras de separação de strings. É global (não por cliente) e só muda com confirmação do usuário; a skill propõe os acréscimos e a CLI valida os arquivos. O usuário aprovou com a condição de ser a melhor forma atual de um agente aprender padrões. Justificativa registrada: o que se aprende aqui são correspondências exatas e auditáveis (Dr.→1, `Nome Fantasia`→`NomFantasia`), em volume pequeno (centenas de entradas), e consumidas por código determinístico. Arquivos estruturados versionados dão precisão, diff legível, revisão humana e testes, sem infraestrutura. Embeddings/banco vetorial servem para recuperar por semelhança em grandes volumes de texto não estruturado: aqui trariam infraestrutura e resultados não determinísticos para um problema de busca exata. A memória do próprio Claude Code (texto livre em CLAUDE.md/memória automática) não é legível pela CLI nem validável. Não existe um padrão único "mais moderno" para isso; essa é a escolha adequada a esse volume e à exigência de auditoria. Busca semântica pode ser acrescentada depois, se o vocabulário crescer muito.
36. **Arquivo de mapeamento YAML por migração** como contrato entre a skill e o núcleo Python. Ele contém: arquivo de origem, aba, tabela externa, tabela de destino, coluna de origem do `IdTmp` e, para cada coluna de destino, a origem e a transformação (renomear, CAST, CASE de códigos, constante). A skill gera o rascunho; a CLI gera o `.sql` a partir dele de forma determinística. Aprovado com a mesma condição e justificativa da entrada 35.
37. **Sugestão de equivalência de colunas em camadas:** primeiro a CLI, de forma determinística (nome normalizado sem acento/caixa, sinônimos da memória, similaridade de texto); depois a skill (Claude), por semântica, para as que sobrarem; o usuário confirma cada uma. O rascunho marca a origem de cada sugestão (memória, similaridade, Claude). Aprovado com a mesma condição e justificativa da entrada 35: o determinístico primeiro deixa o resultado reproduzível e testável, e o LLM fica só onde é preciso entender significado.
38. **Privacidade (LGPD):** a skill só vê cabeçalhos, o perfil das colunas e amostras mascaradas geradas pela CLI, nunca o arquivo bruto.
39. **Dados de cada migração em `workspace/<nome-da-migração>/`**, pasta ignorada pelo git: entradas, CSV limpo, relatórios, plano, mapeamento e `.sql`.
40. **Dry-run em dois passos:** `inspect` gera o relatório (contagem e amostras mascaradas por regra) e um plano de limpeza em YAML com cada regra `approved: false`. O usuário (ou a skill, com confirmação dele) aprova regra a regra, e `apply` executa só as regras aprovadas e gera o CSV limpo (detalha a entrada 21).
41. **Inválidos (CPF/CNPJ/CEP/UF/telefone) decididos por regra, não por linha:** manter o valor original, virar NULL, ou excluir as linhas (só com aprovação explícita). Os inválidos também são exportados para um CSV separado, para revisão manual (detalha a entrada 19).
42. **Duplicatas:** detectar as exatas e as por documento (mesmo CPF/CNPJ). Para as exatas, oferecer remoção com aprovação; as por documento são só relatadas.
43. **Cabeçalhos originais do CSV são mantidos exatamente como vieram**, sem normalização (rejeita a proposta de snake_case). Motivo do usuário: o SQL só faz SELECT da tabela externa e INSERT no destino, então importa o conteúdo e não o nome na origem; manter os nomes originais também evita problemas no mapeamento. O SQL sempre usa crase.
44. **Nome padrão da tabela externa: o nome do arquivo** (`procedencias.csv` → `procedencias`), que o usuário pode sobrescrever. Esse ponto não foi discutido explicitamente; ficou o padrão proposto.
45. **Schema de destino na v1 vem só da DDL**, analisada com `sqlglot`. A conexão somente leitura via `information_schema` fica para um ticket posterior/opcional (detalha a entrada 16).
46. **Um único `.sql` por migração, cobrindo todas as tabelas de destino** (não um por tabela), com cabeçalho comentado para o analista (arquivo de origem, data, nº de linhas, regras aplicadas).
47. **Stack:** Python 3.12+ com `uv`; `polars` em modo streaming/lazy, com todas as colunas lidas como texto; `fastexcel`/`openpyxl` para `.xlsx`; `charset-normalizer` para encoding; `sqlglot` para DDL; `PyYAML`; `Typer` para a CLI; `pytest`.
48. **Uma única skill do Claude Code**, em `.claude/skills/migrar/`, invocada como `/migrar <arquivo>`. Ela conduz o fluxo todo (inspect → aprovação → apply → mapeamento → SQL → atualizar memória) chamando a CLI e entra nos últimos tickets, quando a CLI já funciona sozinha.

Decisões da rodada 3 da entrevista (o usuário aprovou todas as recomendações):

49. **Vocabulário do domínio adotado e registrado em `CONTEXT.md` na raiz:** migração, importação, arquivo de origem, CSV limpo, tabela externa, tabela de destino, ID de origem, equivalência, mapeamento, tabela de códigos, valor de preenchimento, regra de limpeza/separação, inspeção (relatório de inspeção, plano de limpeza), formato conhecido, inválido, memória de padrões, script de migração, analista. O `docs/glossario.md` do curso fica só para os termos técnicos.
50. **Cada arquivo de origem alimenta exatamente uma tabela de destino** na v1. Uma migração pode ter vários arquivos; a ordem dos blocos no script de migração é a ordem do mapeamento (detalha a entrada 46).
51. **`IdTmp` já existente:** se a DDL fornecida já mostrar `IdTmp` na tabela de destino, a CLI omite o `ALTER TABLE ... ADD COLUMN IdTmp` e escreve um aviso comentado no script. Caso contrário, mantém o padrão `ALTER` → `INSERT` → `DROP TABLE` (detalha a entrada 14).
52. **Cabeçalhos inválidos para o MySQL** (vazio, duplicado, mais de 64 caracteres) são sinalizados na inspeção, que propõe uma regra de renomear só aquela coluna, com aprovação. Todos os outros cabeçalhos ficam intactos (detalha a entrada 43).
53. **A inspeção relata valores longos demais:** mais de 500 caracteres (não cabem na tabela externa) e maiores que o tamanho da coluna de destino lido da DDL, com contagem e amostra mascarada.
54. **Valores sem código:** a inspeção conta os valores distintos que cairiam no `ELSE 0` de uma tabela de códigos e oferece incluí-los. Cada inclusão exige confirmação e vira memória de padrões.
55. **A chave primária auto-incremento fica fora do `INSERT`.** O valor de preenchimento é sempre explícito, mesmo quando a coluna tem `DEFAULT` na DDL (detalha as entradas 22 e 29).
56. **Tabelas de código configuradas por um comando reutilizável da CLI** (algo como `codes add <Tabela>.<Coluna>`), também exposto pela skill. Ele pergunta os pares valor → código e grava na memória de padrões, e serve tanto na configuração inicial quanto a qualquer momento depois (detalha a entrada 34).
57. **Granularidade: muitos tickets pequenos**, por volta de 15 a 20 aulas curtas. Cada ticket é uma aula e termina num resultado visível que o usuário roda.
58. **Primeira fatia, logo após a configuração do ambiente: tracer bullet ponta a ponta mínimo.** Ler um CSV com `,`, escrever com `;` e gerar um `INSERT ... SELECT` simples a partir de um mapeamento escrito à mão. Validações, Excel, memória e skill vêm como fatias seguintes.
59. **A conexão somente leitura ao banco é o último ticket, marcado como opcional** (detalha a entrada 45).
60. **Entrevista encerrada.** O usuário confirmou o entendimento compartilhado (fronteira vazia).
61. **Tracker: um board privado do GitHub Projects na conta `William-SWS`, substituindo a entrada 26.** Spec e tickets deixam de ser arquivos locais em `docs/issues/` e vão para esse board, que precisa ser privado. Formato provável, ainda pendente de confirmação do usuário: repositório privado `William-SWS/data-migration-agent` com as issues (spec incluída), ligado a um board Projects (v2) privado. Quem cria é o usuário, pela sessão principal.
