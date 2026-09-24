# Automatização de testes em pipiline de Big Data
Smoke test superficial e rápido que verifica se o sistema "liga" e responde minimamente após deploy/instalação.

## Fluxo em runtime
```
ansible-playbook -i inventory/projeto_madalena playbooks/test_trino.yml
│
├─ 1) ansible.cfg (lido do diretório atual)
│      roles_path=roles · become=sudo/root · ssh pipelining · saída yaml
│
├─ 2) Inventário: inventory/projeto_madalena/hosts.yml
│      grupo [trino] → host madalena (ansible_host=157.230.202.248)
│
├─ 3) Variáveis automáticas: inventory/projeto_madalena/group_vars/all.yml
│      ansible_user, trino_home, trino_port, trino_scheme, validate_certs
│
├─ 4) Playbook: play "hosts: trino" → resolve p/ madalena · become: true
│
├─ 5) Role trino_smoke (achada via roles_path=roles)
│      ├─ defaults/main.yml → knobs: smoke_opa_probe=false, opa_*, probe_*
│      └─ tasks/main.yml → executa via SSH (pipelining), em ordem:
│          1. service_facts                  (systemd da madalena)
│          2. assert  → serviço running/enabled
│          3. wait_for → porta 8443 aberta
│          4. uri GET /v1/info               (na própria madalena, 127.0.0.1)
│          5. assert  → payload coerente (nodeVersion/state)
│          6. slurp → access-control.properties
│          7. assert  → wiring OPA completo (4 propriedades)
│          8. command pgrep → processo Java vivo
│          9. [opcional] probe OPA → delegate_to: localhost (roda NA SUA máquina)
│
└─ 6) Relatório final (callback yaml): OK/FAIL por task + mensagens dos asserts
```

## Execução

```
# Teste completo
ansible-playbook -i inventory/projeto_madalena playbooks/test_trino.yml

# Quando quiser ligar a sonda OPA (sem mudar código):  Ligando a probe OPA sem editar nada.
ansible-playbook -i inventory/projeto_madalena playbooks/test_trino.yml -e smoke_opa_probe=true

# Apenas scripts (via tag)
ansible-playbook -i inventory/projeto_madalena playbooks/run_all_madalena.yml --tags trino
```
