# Automatização de testes em pipiline de Big Data
Smoke test superficial e rápido que verifica se o sistema "liga" e responde minimamente após deploy/instalação.

## Arquitetura

- Docker

## Execução

```
# Teste completo
ansible-playbook -i inventory/projeto_madalena playbooks/test_postgres.yml

# Apenas scripts (via tag)
ansible-playbook -i inventory/projeto_madalena playbooks/run_all_madalena.yml --tags postgres
```