# Migração de dados para MySQL

Vocabulário do domínio do agente que limpa arquivos recebidos (CSV, Excel) e sugere o script SQL que leva esses dados para as tabelas do sistema do usuário, em MySQL.

## O fluxo

**Migração**:
Uma entrega completa: um ou mais arquivos de origem levados para tabelas de destino, resultando em um único script de migração.
_Evitar_: carga, importação

**Importação**:
O passo manual em que o usuário carrega o CSV limpo no MySQL pelo DBeaver, criando a tabela externa.
_Evitar_: migração, carga

**Inspeção**:
A análise de um arquivo de origem sem alterá-lo (dry-run). Produz o relatório de inspeção e o plano de limpeza.
_Evitar_: análise, validação

**Relatório de inspeção**:
O resultado da inspeção: para cada regra de limpeza, quantos registros ela modifica ou deleta, com amostras mascaradas.

**Plano de limpeza**:
A lista das regras de limpeza propostas para um arquivo de origem, cada uma com seu estado de aprovação. Só as regras aprovadas são aplicadas.

**Script de migração**:
O arquivo `.sql` gerado para uma migração, entregue ao analista. Para cada tabela de destino segue o padrão: adiciona `IdTmp`, insere com `INSERT ... SELECT` a partir da tabela externa, remove a tabela externa.
_Evitar_: query, script de importação

**Analista**:
Quem executa o script de migração no banco de produção. O agente nunca executa o script.

## Dados

**Arquivo de origem**:
O arquivo CSV ou Excel recebido, com os dados a migrar.
_Evitar_: base de dados, base, planilha

**CSV limpo**:
O arquivo de origem depois das regras de limpeza aprovadas, no formato que a importação espera.
_Evitar_: CSV tratado, arquivo final

**Tabela externa**:
O CSV limpo já importado no MySQL, com todas as colunas como texto (ex.: `procedencias`, `profissionais`). É removida pelo próprio script de migração.
_Evitar_: tabela temporária, tabela de origem

**Tabela de destino**:
A tabela do sistema do usuário que recebe os dados (ex.: `fatinstituicao`, `Medico`).
_Evitar_: tabela do sistema, tabela final

**IdTmp**:
Coluna temporária adicionada à tabela de destino que guarda o identificador que cada registro tinha na origem.

**ID de origem**:
A coluna do arquivo de origem cujo valor vai para `IdTmp` (ex.: `ID`, `profissional_id`).

## Mapeamento

**Equivalência**:
O par entre uma coluna do arquivo de origem e uma coluna da tabela de destino.
_Evitar_: de-para, correspondência

**Mapeamento**:
O conjunto de equivalências e transformações de uma migração, do qual o script de migração é gerado.

**Tabela de códigos**:
A tradução de valores textuais da origem para os códigos numéricos de uma coluna de destino (ex.: título "Dr." → 1, "Dra." → 2). Valores sem código viram 0.
_Evitar_: de-para, dicionário

**Valor de preenchimento**:
O valor que entra numa coluna de destino que não existe na origem: 0 para colunas que não são data, e uma data fixa (por padrão `2000-01-01`) para colunas de data.
_Evitar_: valor padrão, default

## Limpeza

**Regra de limpeza**:
Uma transformação ou validação aplicada ao arquivo de origem (separador, encoding, data, documento, título...).
_Evitar_: tratamento, filtro

**Regra de separação**:
Uma regra de limpeza que divide um texto em mais de uma coluna, como tirar o título "Dr."/"Dra." do nome de um médico.

**Formato conhecido**:
Um tipo de valor com forma e validação definidas: CPF, CNPJ, CEP, UF, telefone, data.

**Inválido**:
Um valor que falha na validação do seu formato conhecido (ex.: CPF com dígito verificador errado). Sobre ele se decide por regra; nunca é descartado em silêncio.

**Célula vazia**:
Um campo sem valor no arquivo de origem. No CSV limpo vira NULL.
_Evitar_: registro vazio

**Amostra mascarada**:
Um trecho de valores reais em que os dados pessoais estão ocultados, que é a única forma em que os dados chegam ao modelo de linguagem.

## Aprendizado

**Memória de padrões**:
Tudo o que o agente aprendeu em migrações anteriores e reaproveita: sinônimos de colunas, tabelas de código, valores de preenchimento e regras de separação. Só muda com confirmação do usuário.
_Evitar_: histórico, cache

**Sinônimo de coluna**:
Um nome de coluna de origem já visto que corresponde a uma coluna de destino conhecida (ex.: `Nome Fantasia` → `NomFantasia`).
