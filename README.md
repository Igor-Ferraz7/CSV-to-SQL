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

**Geração SQL**
- INSERT - Inserção de novos registros
- UPDATE - Atualização de registros existentes
- DELETE - Remoção de registros com filtros
- SELECT - Consulta com cláusulas WHERE customizáveis
- MERGE - Operação unificada de upsert

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
2. Carregue um arquivo CSV ou arraste-o para a zona de drop
3. Selecione o delimitador (auto-detectado por padrão)
4. Configure o schema e nome da tabela de destino
5. Escolha o tipo de operação SQL (INSERT, UPDATE, etc)
6. Revise e ajuste os tipos de coluna conforme necessário
7. Copie o SQL gerado e execute em seu banco de dados

## Requisitos

- Navegador moderno com suporte a ES6
- Arquivo CSV válido com no mínimo uma coluna

## Estrutura do projeto
