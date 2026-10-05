## run_prestocli.sh (1)
```
#!/bin/bash

#set -e

# Define Java 24 runtime for the Trino CLI
export JAVA_HOME="/home/pentaho/.local/share/mise/installs/java/temurin-24.0.2+12"
export PATH="$JAVA_HOME/bin:$PATH"

source /opt/pentaho/ingestor/shell/.env

export TRINO_PASSWORD=$PRESTO_CLI_PASS

ARQUIVOCONSULTA=$1

echo "VINDO AQUI O ARQUIVO SQL -> $ARQUIVOCONSULTA"

/opt/pentaho/ingestor/shell/trino\
  --server $PRESTO_CLI_SERVER \
  --truststore-path $PRESTO_CLI_JKS \
  --truststore-password $PRESTO_CLI_JKS_PASS \
  --user $PRESTO_CLI_USER \
  --password \
  --file $ARQUIVOCONSULTA 2>&1

if [[ $? -eq 0 ]]; then
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

export TRINO_PASSWORD='CisternaPintinhoInsetoDinossauro'

"/home/trino/.local/share/mise/installs/java/temurin-25/bin/java" -jar /opt/trino/trino_scripts/trino-cli.jar \
  --server https://madalena.fabricadenegocio.com.br:8443 \
  --user trino \
  --truststore-path /opt/trino/trino-server-482/etc/truststore.jks \
  --truststore-password NetoAnticongelanteCadeiaCatalunha \
  --password \
  "$@"

unset TRINO_PASSWORD
```

## shelldelete.sh (3)
```
#!/bin/bash

set -e

DIR_TMP_DATA=/var/data/hadoop/dados

echo 'Deletando arquivos' 

rm -r $DIR_TMP_DATA/*.sql
rm -r $DIR_TMP_DATA/*.csv

echo 'Arquivos Deletados'
```

## shellhadoop.sh (4)
```
#!/bin/bash

set -e

FILENAME=$1
DIRHADOOP=$2

HDFS_CMD_PATH=/usr/local/hadoop/hadoop/bin

echo "Inserindo arquivo -> $FILENAME"

$HDFS_CMD_PATH/hdfs dfs -mkdir -p $DIRHADOOP

$HDFS_CMD_PATH/hdfs dfs -put $FILENAME $DIRHADOOP

echo "Arquivo $FILENAME Inserido em $DIRHADOOP"
```

## shellhadoopdelete.sh (5)
```
#!/bin/bash

FILENAME=$1
HDFS_LOCATION=$2
echo $FILENAME
echo $HDFS_LOCATION
HDFS_CMD_PATH=/usr/local/hadoop/hadoop/bin

echo 'Deletando arquivo -> ' $FILENAME

# Checa se arquivo existe no hadoop
if $HDFS_CMD_PATH/hdfs dfs -test -e $FILENAME; then

  # Deleta o arquivo para sempre
  $HDFS_CMD_PATH/hdfs dfs -rm -r -skipTrash $FILENAME

  echo "Diretorio e Arquivo Deletado -> $FILENAME"
  exit 0
else
  echo "Diretorio ou Arquivo não existe: nada foi feito."
  exit 0
fi
```

## shellhadoopdeleten.sh (6)
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
FILENAME2=$2
FILENAME3=$3
FILENAME4=$4

echo 'FILENAME' $FILENAME
echo 'FILENAME2' $FILENAME2
echo 'FILENAME3' $FILENAME3
echo 'FILENAME4' $FILENAME4



echo 'Deletando arquivo -> ' $FILENAME

/usr/local/hadoop/hadoop/bin/hadoop fs -rm -r $FILENAME

echo 'Diretorio e Arquivo Deletado'
```

## shellhadooprollbackcreate.sh (8)
```
#!/bin/bash

#set -e

CONJUNTO=$1
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
    $HDFS_CMD_PATH/hdfs dfs -mkdir -p "$ROLLBACK_PATH"
  fi

  # Cria pasta rollback conjunto
  if ! $HDFS_CMD_PATH/hdfs dfs -test -e "$ROLLBACK_CONJUNTO_PATH"; then
    echo "Criando pasta de rollback para o conjunto $CONJUNTO em $ROLLBACK_CONJUNTO_PATH"
    $HDFS_CMD_PATH/hdfs dfs -mkdir -p "$ROLLBACK_CONJUNTO_PATH"
  fi

  # Checa se existe arquivos em rollback
  $HDFS_CMD_PATH/hdfs dfs -test -e "$ROLLBACK_CONJUNTO_PATH/*" 2> /dev/null

  if [ $? -eq 0 ]; then
    $HDFS_CMD_PATH/hdfs dfs -rm "$ROLLBACK_CONJUNTO_PATH/*" 2> /dev/null
  fi

  $HDFS_CMD_PATH/hdfs dfs -test -e "$CONJUNTO_PATH/*" 2> /dev/null

  # Verifica se pasta tem arquivo e mv caso verdadeiro
  if [ $? -eq 0 ]; then
    echo "Movendo arquivos do conjunto $CONJUNTO para $ROLLBACK_CONJUNTO_PATH"
    $HDFS_CMD_PATH/hdfs dfs -cp "$CONJUNTO_PATH/*" "$ROLLBACK_CONJUNTO_PATH/"
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

#set -e

CONJUNTO=$1
HDFS_CMD_PATH=/usr/local/hadoop/hadoop/bin
DADO_BRUTO_PATH="/zonas/dado_bruto"
ROLLBACK_PATH="/zonas/rollback"

if [ -z "$CONJUNTO" ]; then
  echo "ERRO: Nenhum conjunto foi especificado."
  exit 1
fi

CONJUNTO_PATH="$DADO_BRUTO_PATH/$CONJUNTO"
ROLLBACK_CONJUNTO_PATH="$ROLLBACK_PATH/$CONJUNTO"

$HDFS_CMD_PATH/hdfs dfs -test -e "$ROLLBACK_CONJUNTO_PATH/*" 2> /dev/null

if [ $? -eq 0 ]; then
  echo "Rollback encontrado para o conjunto $CONJUNTO em $ROLLBACK_CONJUNTO_PATH"

  echo "Limpando pasta $CONJUNTO_PATH antes do rollback"

  $HDFS_CMD_PATH/hdfs dfs -test -e "$CONJUNTO_PATH/*" 2> /dev/null
 
  if [ $? -eq 0 ]; then
    $HDFS_CMD_PATH/hdfs dfs -rm "$CONJUNTO_PATH/*" 2> /dev/null
  else
    echo "INFO: Pasta $CONJUNTO_PATH vazia. Nada feito"
  fi

  echo "Restaurando arquivos do rollback para $CONJUNTO_PATH"
  $HDFS_CMD_PATH/hdfs dfs -mv "$ROLLBACK_CONJUNTO_PATH/*" "$CONJUNTO_PATH/"
   
  echo "Rollback realizado com sucesso. Arquivos restaurados para $CONJUNTO_PATH"
else
  echo "INFO: Nenhum rollback encontrado para o conjunto $CONJUNTO em $ROLLBACK_PATH."
  exit 0
fi
```

## shellhadooprollbackremove.sh (10)
```
#!/bin/bash

#set -e

CONJUNTO=$1
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

  $HDFS_CMD_PATH/hdfs dfs -rm -r "$ROLLBACK_CONJUNTO_PATH" 2> /dev/null

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
LOG_FILE="/var/lib/pdi/ingestor/shell/shell.log"
HDFS_CMD_PATH="/usr/local/hadoop/hadoop/bin"

if [ -z "$NOME_CONJUNTO" ]; then
  echo "ERRO: Nenhum nome de conjunto especificado." >> $LOG_FILE 2>&1
  exit 64
fi

echo "Removendo e recriando diretório final HDFS: ${FINAL_DIR}"
$HDFS_CMD_PATH/hdfs dfs -rm -r -f "${FINAL_DIR}" >> $LOG_FILE 2>&1

if [ $? -ne 0 ]; then
    echo "ERRO CRÍTICO: Falha ao tentar remover o diretório HDFS: ${FINAL_DIR}." >> $LOG_FILE 2>&1
    exit 65
fi

echo "Criando diretório final HDFS: ${FINAL_DIR}"
$HDFS_CMD_PATH/hdfs dfs -Dfs.permissions.umask-mode=002 -mkdir -p "${FINAL_DIR}" >> $LOG_FILE 2>&1

if [ $? -ne 0 ]; then
    echo "ERRO CRÍTICO: Falha ao tentar criar o diretório HDFS: ${FINAL_DIR}" >> $LOG_FILE 2>&1
    exit 65
fi

echo "Diretório final pronto: ${FINAL_DIR}"


echo "ANTES DE MOVER"
$HDFS_CMD_PATH/hdfs dfs -ls /zonas/tmp


tentativas=0
max_tentativas=12 # 60 segundos timeout
sleep_interval=5

echo "Aguardando arquivos aparecerem em ${TMP_DIR} (Timeout: 60s)..."

while true; do
  if $HDFS_CMD_PATH/hdfs dfs -ls -d "${TMP_DIR}"/* > /dev/null 2>&1; then
    break # Arquivos encontrados -> break loop
  fi

  tentativas=$((tentativas + 1))
  if [ $tentativas -ge $max_tentativas ]; then
    echo "ERRO: Timeout esperando arquivos no diretório temporário (${TMP_DIR})." >> $LOG_FILE 2>&1
    exit 1
  fi

  echo "Tentativa $tentativas de $max_tentativas. Aguardando ${sleep_interval}s..."
  sleep $sleep_interval
done

echo "Iniciando movimentação de arquivos..."

$HDFS_CMD_PATH/hdfs dfs -ls "${TMP_DIR}" | awk 'NR > 1 {print $8}' | while IFS= read -r file_path; do

  if [ "$file_path" == "$TMP_DIR" ]; then
    continue
  fi

  filename=$(basename "$file_path")

  echo "Movendo arquivo: $filename para ${FINAL_DIR}/"
  $HDFS_CMD_PATH/hdfs dfs -Dfs.permissions.umask-mode=002 -mv "$file_path" "${FINAL_DIR}/${filename}" >> $LOG_FILE 2>&1

  if [ $? -ne 0 ]; then
    echo "ERRO: Falha ao mover arquivo: $filename. O script continuará, mas o status final será de erro." >> $LOG_FILE 2>&1
    MOVE_FALHOU=1
  fi

done

echo "Removendo diretório temporário: ${TMP_DIR}"
#$HDFS_CMD_PATH/hdfs dfs -rm -r -f "${TMP_DIR}" >> $LOG_FILE 2>&1

if [ $MOVE_FALHOU -eq 1 ]; then
  echo "AVISO: Alguns arquivos falharam na movimentação. Verifique os logs." >> $LOG_FILE 2>&1
  exit 1
else
  echo "Sucesso: Todos os arquivos foram movidos e sobrescritos com sucesso."
  exit 0
fi
```
