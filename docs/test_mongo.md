# Automatização de testes em pipiline de Big Data
Smoke test superficial e rápido que verifica se o sistema "liga" e responde minimamente após deploy/instalação.


## Arquitetura

- Docker

## 🔐 Senha do Mongo (somente se mongo_test_auth: true)

## Caso seja necessária a utilização do vault
```
cd BIGDATA-INFRA-TESTS
mkdir -p inventory/projeto_madalena/group_vars/mongo
ansible-vault create inventory/projeto_madalena/group_vars/mongo/vault.yml
# dentro: vault_mongo_admin_password: <senha>

# executar passando o vault:
ansible-playbook -i inventory/projeto_madalena playbooks/test_mongo.yml --ask-vault-pass
```

## Execução

```
# Teste completo
ansible-playbook -i inventory/projeto_madalena playbooks/test_mongo.yml

# Apenas scripts (via tag)
ansible-playbook -i inventory/projeto_madalena playbooks/run_all_madalena.yml --tags mongo
```