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
ansible-playbook -i inventory/projeto_madalena playbooks/test_mongo.yml
```