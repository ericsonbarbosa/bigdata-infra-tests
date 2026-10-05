## run_prestocli.sh (1)
```
#!/bin/bash

# Define Java 21 runtime para o Trino CLI (runtime de sistema, gerenciado pelo SO)
export JAVA_HOME="/usr/lib/jvm/java-21-openjdk-amd64"
export PATH="$JAVA_HOME/bin:$PATH"

source /opt/pentaho/ingestor/shell/.env

export TRINO_PASSWORD="$PRESTO_CLI_PASS"

ARQUIVOCONSULTA="$1"

if [ -z "$ARQUIVOCONSULTA" ]; then
  echo "ERRO: Nenhum arquivo SQL foi especificado."
  exit 1
fi

echo "VINDO AQUI O ARQUIVO SQL -> $ARQUIVOCONSULTA"

/opt/pentaho/ingestor/shell/trino \
  --server "$PRESTO_CLI_SERVER" \
  --truststore-path "$PRESTO_CLI_JKS" \
  --truststore-password "$PRESTO_CLI_JKS_PASS" \
  --user "$PRESTO_CLI_USER" \
  --password \
  --file "$ARQUIVOCONSULTA" 2>&1
RC=$?

if [ $RC -eq 0 ]; then
  echo "CONSULTA EXECUTADA COM SUCESSO"
  exit 0
else
  echo "CONSULTA COM ERRO" 1>&2
  exit 1
fi
```

## run_shell.sh (2)
```
#!/bin/bash

source /opt/pentaho/ingestor/shell/.env

export TRINO_PASSWORD="$PRESTO_CLI_PASS"

JAVA_BIN="/usr/lib/jvm/java-21-openjdk-amd64/bin/java"
TRINO_JAR="/opt/trino/trino_scripts/trino-cli.jar"

"$JAVA_BIN" -jar "$TRINO_JAR" \
  --server "$PRESTO_CLI_SERVER" \
  --user "$PRESTO_CLI_USER" \
  --truststore-path "$PRESTO_CLI_JKS" \
  --truststore-password "$PRESTO_CLI_JKS_PASS" \
  --password \
  "$@"
RC=$?

unset TRINO_PASSWORD
exit $RC
```

## shelldelete.sh (3)
```
#!/bin/bash
set -euo pipefail

DIR_TMP_DATA=/var/data/hadoop/dados

if [ ! -d "$DIR_TMP_DATA" ]; then
  echo "ERRO: diretório $DIR_TMP_DATA não existe."
  exit 1
fi

echo 'Deletando arquivos'

# nullglob: sem matches o array fica vazio (em vez de manter o padrão literal)
shopt -s nullglob
arquivos=("$DIR_TMP_DATA"/*.sql "$DIR_TMP_DATA"/*.csv)
shopt -u nullglob

if [ ${#arquivos[@]} -eq 0 ]; then
  echo 'Nenhum arquivo para deletar.'
  echo 'Arquivos Deletados'
  exit 0
fi

rm -r "${arquivos[@]}"

echo 'Arquivos Deletados'
```

## shellhadoop.sh (4)
```
#!/bin/bash
set -euo pipefail

FILENAME="${1:-}"
DIRHADOOP="${2:-}"

HDFS_CMD_PATH=/usr/local/hadoop/hadoop/bin

if [ -z "$FILENAME" ] || [ -z "$DIRHADOOP" ]; then
  echo "ERRO: uso: shellhadoop.sh <arquivo_local> <diretorio_hdfs>"
  exit 1
fi

if [ ! -f "$FILENAME" ]; then
  echo "ERRO: arquivo local não encontrado: $FILENAME"
  exit 1
fi

echo "Inserindo arquivo -> $FILENAME"

$HDFS_CMD_PATH/hdfs dfs -mkdir -p "$DIRHADOOP"

$HDFS_CMD_PATH/hdfs dfs -put "$FILENAME" "$DIRHADOOP"

echo "Arquivo $FILENAME Inserido em $DIRHADOOP"
```

## shellhadoopdelete.sh (5)
```
#!/bin/bash

FILENAME="$1"
HDFS_CMD_PATH=/usr/local/hadoop/hadoop/bin

if [ -z "$FILENAME" ]; then
  echo "Nenhum arquivo informado em FILENAME."
  exit 0
fi

# Suporta lista separada por vírgulas (contrato da versão de referência)
IFS=',' read -r -a FILES <<< "$FILENAME"

for file in "${FILES[@]}"; do
  trimmed="$(echo "$file" | xargs)"
  [ -n "$trimmed" ] || continue

  echo "Deletando arquivo -> $trimmed"

  if $HDFS_CMD_PATH/hdfs dfs -test -e "$trimmed"; then
    if $HDFS_CMD_PATH/hdfs dfs -rm -r -skipTrash "$trimmed"; then
      echo "Diretorio e Arquivo Deletado -> $trimmed"
    else
      echo "ERRO: falha ao remover $trimmed"
      exit 1
    fi
  else
    echo "Diretorio ou Arquivo não existe: nada foi feito."
  fi
done

exit 0
```

## shellhadoopdeleten.sh --> DEPRECADO (renomeado .bkp em produção e removido da esteira de testes) (6)
```
#!/bin/bash
FILENAME=$1/*
FILENAME2=$2/*

#echo 'Deletando arquivo -> ' $FILENAME
echo 'Deletando arquivo -> ' $FILENAME2
/usr/local/hadoop/hadoop-2.7.7/bin/hadoop fs -rm -r -skipTrash $FILENAME2
#hdfs dfs -rm -R -skipTrash $FILENAME
```

## shellhadooptruncate.sh (7)
```
#!/bin/bash

FILENAME=$1
HDFS_CMD_PATH=/usr/local/hadoop/hadoop/bin

if [ -z "$FILENAME" ]; then
  echo "ERRO: Nenhum caminho foi especificado."
  exit 1
fi

echo 'Deletando arquivo -> ' "$FILENAME"

# Checa se o caminho existe no HDFS
if $HDFS_CMD_PATH/hdfs dfs -test -e "$FILENAME"; then
  if $HDFS_CMD_PATH/hdfs dfs -rm -r "$FILENAME"; then
    echo 'Diretorio e Arquivo Deletado'
    exit 0
  else
    echo "ERRO: Falha ao remover $FILENAME"
    exit 1
  fi
else
  echo "Diretorio ou Arquivo não existe: nada foi feito."
  exit 0
fi
```

## shellhadooprollbackcreate.sh (8)
```
#!/bin/bash

CONJUNTO="$1"
HDFS_CMD_PATH=/usr/local/hadoop/hadoop/bin
DADO_BRUTO_PATH="/zonas/dado_bruto"
ROLLBACK_PATH="/zonas/rollback"

if [ -z "$CONJUNTO" ]; then
  echo "ERRO: Nenhum conjunto foi especificado."
  exit 1
fi

CONJUNTO_PATH="$DADO_BRUTO_PATH/$CONJUNTO"
ROLLBACK_CONJUNTO_PATH="$ROLLBACK_PATH/$CONJUNTO"

# Verifica se o conjunto existe
if $HDFS_CMD_PATH/hdfs dfs -test -e "$CONJUNTO_PATH"; then
  echo "Conjunto $CONJUNTO encontrado em $CONJUNTO_PATH"

  # Cria pasta rollback
  if ! $HDFS_CMD_PATH/hdfs dfs -test -e "$ROLLBACK_PATH"; then
    echo "Criando pasta de rollback em $ROLLBACK_PATH"
    $HDFS_CMD_PATH/hdfs dfs -mkdir -p "$ROLLBACK_PATH" || { echo "ERRO: falha ao criar $ROLLBACK_PATH"; exit 1; }
  fi

  # Cria pasta rollback conjunto
  if ! $HDFS_CMD_PATH/hdfs dfs -test -e "$ROLLBACK_CONJUNTO_PATH"; then
    echo "Criando pasta de rollback para o conjunto $CONJUNTO em $ROLLBACK_CONJUNTO_PATH"
    $HDFS_CMD_PATH/hdfs dfs -mkdir -p "$ROLLBACK_CONJUNTO_PATH" || { echo "ERRO: falha ao criar $ROLLBACK_CONJUNTO_PATH"; exit 1; }
  fi

  # Checa se existe arquivos em rollback e limpa
  if $HDFS_CMD_PATH/hdfs dfs -test -e "$ROLLBACK_CONJUNTO_PATH/*" 2> /dev/null; then
    $HDFS_CMD_PATH/hdfs dfs -rm "$ROLLBACK_CONJUNTO_PATH/*" 2> /dev/null || { echo "ERRO: falha ao limpar rollback antigo de $CONJUNTO"; exit 1; }
  fi

  # Verifica se pasta tem arquivo e faz o backup
  if $HDFS_CMD_PATH/hdfs dfs -test -e "$CONJUNTO_PATH/*" 2> /dev/null; then
    echo "Movendo arquivos do conjunto $CONJUNTO para $ROLLBACK_CONJUNTO_PATH"
    $HDFS_CMD_PATH/hdfs dfs -cp "$CONJUNTO_PATH/*" "$ROLLBACK_CONJUNTO_PATH/" || { echo "ERRO: falha no backup de $CONJUNTO"; exit 1; }
  else
    echo "Pasta $CONJUNTO_PATH vazia. Nada feito"
    exit 0
  fi

  echo "Backup realizado com sucesso para rollback em $ROLLBACK_CONJUNTO_PATH"
else
  # Caso o conjunto não exista
  echo "INFO: Pasta do $CONJUNTO não existe. Nada feito."
  exit 0
fi
```

## shellhadooprollbackrestore.sh (9)
```
#!/bin/bash

CONJUNTO="$1"
HDFS_CMD_PATH=/usr/local/hadoop/hadoop/bin
DADO_BRUTO_PATH="/zonas/dado_bruto"
ROLLBACK_PATH="/zonas/rollback"

if [ -z "$CONJUNTO" ]; then
  echo "ERRO: Nenhum conjunto foi especificado."
  exit 1
fi

CONJUNTO_PATH="$DADO_BRUTO_PATH/$CONJUNTO"
ROLLBACK_CONJUNTO_PATH="$ROLLBACK_PATH/$CONJUNTO"

if $HDFS_CMD_PATH/hdfs dfs -test -e "$ROLLBACK_CONJUNTO_PATH/*" 2> /dev/null; then
  echo "Rollback encontrado para o conjunto $CONJUNTO em $ROLLBACK_CONJUNTO_PATH"

  echo "Limpando pasta $CONJUNTO_PATH antes do rollback"
  if $HDFS_CMD_PATH/hdfs dfs -test -e "$CONJUNTO_PATH/*" 2> /dev/null; then
    $HDFS_CMD_PATH/hdfs dfs -rm "$CONJUNTO_PATH/*" 2> /dev/null || { echo "ERRO: falha ao limpar $CONJUNTO_PATH"; exit 1; }
  else
    echo "INFO: Pasta $CONJUNTO_PATH vazia. Nada feito"
  fi

  echo "Restaurando arquivos do rollback para $CONJUNTO_PATH"
  $HDFS_CMD_PATH/hdfs dfs -mv "$ROLLBACK_CONJUNTO_PATH/*" "$CONJUNTO_PATH/" || { echo "ERRO: falha ao restaurar $CONJUNTO"; exit 1; }

  echo "Rollback realizado com sucesso. Arquivos restaurados para $CONJUNTO_PATH"
else
  echo "INFO: Nenhum rollback encontrado para o conjunto $CONJUNTO em $ROLLBACK_PATH."
  exit 0
fi
```

## shellhadooprollbackremove.sh (10)
```
#!/bin/bash

CONJUNTO="$1"
HDFS_CMD_PATH=/usr/local/hadoop/hadoop/bin
DADO_BRUTO_PATH="/zonas/dado_bruto"
ROLLBACK_PATH="/zonas/rollback"

if [ -z "$CONJUNTO" ]; then
  echo "ERRO: Nenhum conjunto foi especificado."
  exit 1
fi

CONJUNTO_PATH="$DADO_BRUTO_PATH/$CONJUNTO"
ROLLBACK_CONJUNTO_PATH="$ROLLBACK_PATH/$CONJUNTO"

if $HDFS_CMD_PATH/hdfs dfs -test -e "$ROLLBACK_CONJUNTO_PATH"; then
  $HDFS_CMD_PATH/hdfs dfs -rm -r "$ROLLBACK_CONJUNTO_PATH" 2> /dev/null || { echo "ERRO: falha ao remover rollback de $CONJUNTO"; exit 1; }
  echo "Rollback removido com sucesso."
else
  # Caso não exista rollback para o conjunto
  echo "INFO: Nenhum rollback encontrado para o conjunto $CONJUNTO em $ROLLBACK_PATH. Nada feito"
  exit 0
fi
```

## shellhadooppipelinemove.sh (11)
```
#!/bin/bash

NOME_CONJUNTO="$1"
TMP_DIR="/zonas/tmp/tmp_${NOME_CONJUNTO}"
FINAL_DIR="/zonas/dado_bruto/${NOME_CONJUNTO}"
MOVE_FALHOU=0
LOG_FILE="/opt/pentaho/ingestor/shell/shell.log"
HDFS_CMD_PATH="/usr/local/hadoop/hadoop/bin"

if [ -z "$NOME_CONJUNTO" ]; then
  echo "ERRO: Nenhum nome de conjunto especificado." >> "$LOG_FILE" 2>&1
  exit 64
fi

echo "Removendo e recriando diretório final HDFS: ${FINAL_DIR}"
$HDFS_CMD_PATH/hdfs dfs -rm -r -f "${FINAL_DIR}" >> "$LOG_FILE" 2>&1

if [ $? -ne 0 ]; then
    echo "ERRO CRÍTICO: Falha ao tentar remover o diretório HDFS: ${FINAL_DIR}." >> "$LOG_FILE" 2>&1
    exit 65
fi

echo "Criando diretório final HDFS: ${FINAL_DIR}"
$HDFS_CMD_PATH/hdfs dfs -Dfs.permissions.umask-mode=002 -mkdir -p "${FINAL_DIR}" >> "$LOG_FILE" 2>&1

if [ $? -ne 0 ]; then
    echo "ERRO CRÍTICO: Falha ao tentar criar o diretório HDFS: ${FINAL_DIR}" >> "$LOG_FILE" 2>&1
    exit 65
fi

echo "Diretório final pronto: ${FINAL_DIR}"

tentativas=0
max_tentativas=12   # 12 x 5s = 60s de timeout
sleep_interval=5

echo "Aguardando arquivos aparecerem em ${TMP_DIR} (Timeout: 60s)..."

while true; do
  if $HDFS_CMD_PATH/hdfs dfs -ls -d "${TMP_DIR}"/* > /dev/null 2>&1; then
    break # Arquivos encontrados -> break loop
  fi

  tentativas=$((tentativas + 1))
  if [ $tentativas -ge $max_tentativas ]; then
    echo "ERRO: Timeout esperando arquivos no diretório temporário (${TMP_DIR})." >> "$LOG_FILE" 2>&1
    exit 1
  fi

  echo "Tentativa $tentativas de $max_tentativas. Aguardando ${sleep_interval}s..."
  sleep $sleep_interval
done

echo "Iniciando movimentação de arquivos..."

while IFS= read -r file_path; do

  if [ "$file_path" == "$TMP_DIR" ]; then
    continue
  fi

  filename=$(basename "$file_path")

  echo "Movendo arquivo: $filename para ${FINAL_DIR}/"
  $HDFS_CMD_PATH/hdfs dfs -Dfs.permissions.umask-mode=002 -mv "$file_path" "${FINAL_DIR}/${filename}" >> "$LOG_FILE" 2>&1

  if [ $? -ne 0 ]; then
    echo "ERRO: Falha ao mover arquivo: $filename. O script continuará, mas o status final será de erro." >> "$LOG_FILE" 2>&1
    MOVE_FALHOU=1
  fi

done < <($HDFS_CMD_PATH/hdfs dfs -ls "${TMP_DIR}" | awk 'NR > 1 {print $8}')

echo "Removendo diretório temporário: ${TMP_DIR}"
#$HDFS_CMD_PATH/hdfs dfs -rm -r -f "${TMP_DIR}" >> "$LOG_FILE" 2>&1

if [ $MOVE_FALHOU -eq 1 ]; then
  echo "AVISO: Alguns arquivos falharam na movimentação. Verifique os logs." >> "$LOG_FILE" 2>&1
  exit 1
else
  echo "Sucesso: Todos os arquivos foram movidos e sobrescritos com sucesso."
  exit 0
fi
```
