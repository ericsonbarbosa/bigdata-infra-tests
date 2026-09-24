# Automatização de testes em pipiline de Big Data
Smoke test superficial e rápido que verifica se o sistema "liga" e responde minimamente após deploy/instalação.

## Arquitetura de Configuração do Trino

A Madalena utiliza o padrão "Configuration Directory Pattern":
- `/etc/trino/` é a FONTE DA VERDADE de todas as configurações
- `/opt/trino/trino-server-*/etc` são SYMLINKS para `/etc/trino/`
- Isso permite que múltiplas versões do Trino compartilhem a mesma configuração
- O script `manage_users.sh` deve SEMRE apontar para `/etc/trino/credentials.db`

## Arquivos Legados
- `/etc/presto/credentials.db` é um arquivo descontinuado da era Presto
- NÃO deve ser usado como referência

## Local do Arquivo
`/opt/trino/trino_scripts/manage_users.sh`

## Como Rodar
```
# Teste completo
ansible-playbook -i inventory/projeto_madalena playbooks/test_trino_scripts.yml

# Apenas scripts (via tag)
ansible-playbook -i inventory/projeto_madalena playbooks/run_all_madalena.yml --tags trino_scripts
```
