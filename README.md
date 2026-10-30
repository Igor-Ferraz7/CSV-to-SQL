# CSV-to-SQL

Conversor de arquivos CSV para comandos SQL. Permite transformar dados tabulares em operações SQL executáveis com suporte a múltiplos delimitadores, mapeamento de tipos e variações de operações (INSERT, UPDATE, DELETE, SELECT, MERGE).

## Motivação

Processamento manual de arquivos CSV para SQL é repetitivo, propenso a erros e consome tempo. Este projeto elimina essa etapa fornecendo uma interface onde você pode:

- Importar dados em diferentes formatos (CSV, TSV, TXT)
- Mapear tipos de dados automaticamente ou customizar manualmente
- Gerar SQL para diferentes operações em um único workflow
- Salvar configurações e reutilizá-las em conversões futuras

## Características

**Importação de dados**
- Suporte a CSV, TSV e TXT
- Auto-detecção de delimitadores (vírgula, ponto-e-vírgula, tab, pipe)
- Detecção automática de codificação (UTF-8, Latin-1)
- Preview dos dados antes da conversão

**Excel (XLSX / XLS / ODS) com várias abas**
- Cada aba vira uma tabela, com mapeamento de colunas próprio
- Nome da tabela sugerido a partir do nome da aba (editável)
- Geração de um único script SQL com todas as abas marcadas
- Linha do cabeçalho configurável por aba (para planilhas com título no topo)
- Datas do Excel convertidas para `yyyy-mm-dd` / `yyyy-mm-dd hh:mm:ss`, células mescladas preenchidas
- Config salva por arquivo Excel: ao gerar o SQL, guarda todas as abas (marcadas, tabela de destino, linha do cabeçalho, mapeamento, funções e colunas manuais) e as opções; ao abrir o mesmo arquivo de novo, um aviso oferece **↩ Restaurar** tudo de uma vez (abas casadas pelo nome, colunas pelo cabeçalho)
- Leitura feita com [SheetJS](https://sheetjs.com), carregado sob demanda via CDN (requer internet)

**Geração SQL**
- INSERT - Inserção de novos registros
- UPDATE - Atualização de registros existentes
- DELETE - Remoção de registros com filtros
- SELECT - Consulta com cláusulas WHERE customizáveis
- MERGE - Operação unificada de upsert

**Sequencial a partir do banco**
- A função 🔢 Sequencial (em coluna da planilha ou coluna manual) pode continuar do `MAX(coluna)` da tabela de destino
- O valor é lido no banco numa variável (`DECLARE @seq_... = (SELECT ISNULL(MAX(col), inicio - 1) ...)`) e cada linha recebe `@seq_... + n`
- Abas que vão para a mesma tabela compartilham a numeração, sem pular nem repetir; depois de cada `GO` o MAX é relido
- Em transação, o MAX é lido com `WITH (UPDLOCK, HOLDLOCK)` para evitar colisão com cargas simultâneas
- Vale só para INSERT (nas demais operações usa o valor inicial fixo, com aviso no topo do SQL)

**ROLLBACK automático em caso de erro** (ligado por padrão)
- `TRY/CATCH` em cada lote; antes de inserir, verifica se as tabelas e colunas de destino existem
- Transação **uma por aba** (padrão: `COMMIT` a cada aba bem-sucedida; uma aba com erro é desfeita e as seguintes continuam) ou **uma para tudo** (qualquer erro desfaz todas as abas)
- Em erro: `ROLLBACK` da transação, mensagem com número do erro, linha do script, aba/tabela/lote e o texto do SQL Server; os lotes restantes daquela transação são pulados
- O `COMMIT` só acontece se todos os lotes da transação rodaram — um lote que falhou na compilação também a desfaz
- No fim, um resumo: quantas transações foram gravadas e quantas desfeitas

**Campos obrigatórios e linhas ignoradas** (INSERT / MERGE)
- Marque colunas como **Obrig.** no mapeamento
- Com "Ignorar linhas com campo obrigatório vazio" (padrão ligado), linhas em que algum campo obrigatório sairia `NULL` não entram no SQL — o sequencial não pula números por causa delas
- As linhas ignoradas aparecem na aba **🚫 Ignoradas** (linha no arquivo, campos vazios e os dados originais) e podem ser exportadas como planilha `.xlsx`, uma aba por aba de origem
- Com a opção desligada, as linhas vão para o SQL e um aviso no topo do script informa quantas têm campo obrigatório vazio

**Regras de referência (validação no banco, apenas INSERT)**
- Botão 🔗 na coluna: tabela e coluna de referência, condição SQL opcional e motivo — ex.: `produto` → `Materiais.Codigo` com a condição `Inativo = 'S'`. Três modos de "ignorar a linha quando o registro…":
  - **for encontrado com a condição** — barra só o inativo
  - **não for encontrado, OU for encontrado com a condição** — barra o inexistente **e** o inativo (NULL na condição conta como "não atende", ex.: `Inativo` NULL = ativo)
  - **não for encontrado com a condição** — aqui a condição descreve o registro **válido** (ex.: `Ativo = 'S'`)
- O modal explica em português o que é ignorado e o que entra, e avisa quando a regra parece invertida (ex.: "não for encontrado" + `Inativo = 'S'`, que deixaria entrar só os inativos) — com botão para trocar para o modo certo; regras salvas assim ficam com o 🔗 laranja e geram aviso no topo do SQL
- Excel com várias abas: ao aplicar a regra, o gerador procura o mesmo campo nas outras abas que vão para a mesma tabela (sem diferenciar maiúsculas) e pergunta se quer replicar a regra nelas — dá para escolher as abas; abas com outra regra são avisadas antes de substituir
- Validado na hora da carga: cada bloco vai uma vez para uma tabela temporária, as linhas barradas são registradas e as demais entram num único `INSERT … SELECT` — uma busca na tabela de referência por bloco, não por linha (use índice na coluna de referência)
- O sequencial MAX é numerado depois do filtro (`ROW_NUMBER()`), sem buracos
- Linha barrada não é erro: a transação segue e faz `COMMIT`. No fim do script, um `SELECT` lista as barradas (Aba, Tabela, Linha, Motivo, Coluna, Valor)
- Copie esse resultado no SSMS e use **📥 Importar do banco** na aba 🚫 Ignoradas: as linhas são casadas com o arquivo carregado e entram na mesma planilha exportada
- Se uma transação sofrer ROLLBACK por outro erro, as linhas que ela barrou também saem da lista (a aba inteira precisa ser recarregada de qualquer forma)
- Requer SQL Server 2012+ e execução no SSMS/sqlcmd (usa `GO`)

**Mapeamento de colunas**
- Detecção automática de tipos (INT, VARCHAR, DATETIME, DECIMAL, BIT)
- Ajuste manual de tipos e nomes de colunas
- Suporte a schemas customizados
- Validação de tipos antes da geração

**Gerenciamento de configurações**
- Salve perfis de mapeamento para reuso
- Histórico de tabelas convertidas
- Restauração automática de configurações prévias
- Exportação e importação de perfis

## Como usar

1. Acesse a ferramenta no navegador
2. Carregue um arquivo CSV ou Excel ou arraste-o para a zona de drop
3. CSV: selecione o delimitador (auto-detectado por padrão). Excel: marque as abas a gerar e clique em cada uma para configurá-la
4. Configure o schema e nome da tabela de destino (no Excel, uma tabela por aba)
5. Escolha o tipo de operação SQL (INSERT, UPDATE, etc)
6. Revise e ajuste os tipos de coluna conforme necessário
7. Copie o SQL gerado e execute em seu banco de dados

## Requisitos

- Navegador moderno com suporte a ES6
- Arquivo CSV ou Excel válido com no mínimo uma coluna

## Estrutura do projeto
