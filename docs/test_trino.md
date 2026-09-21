# bigdata-infra-tests-
Smoke test superficial e rápido que verifica se o sistema "liga" e responde minimamente após deploy/instalação.


## Estrutura

```
bigdata-infra-tests/            ← repo dedicado aos testes (Git separado)
├── ansible.cfg                 ← "COMO o Ansible se comporta" (config global do repo)
├── README.md
│
├── roles/                      ← "COMO" (lógica reutilizável entre projetos)
│   └── trino_smoke/            ← role reutilizável (lógica dos testes)
│       ├── tasks/
│       │   └── main.yml        ← passos do teste, em ordem
│       └── defaults/
│           └── main.yml        ← valores default (porta, schema, etc.)
│
├── inventory/                  ← "CONTRA QUEM" (1 diretório por projeto/ambiente)
│   ├── projeto_madalena/
│   │   ├── hosts.yml           ← hosts e grupos: grupo [trino] → madalena (IP) | madalena:8443, goodview:8282, etc.
│   │   └── group_vars/
│   │       └── all.yml         ← variáveis DO PROJETO (sobrepõem defaults da role)
│   ├── projeto_b/
│   │   ├── hosts.yml
│   │   └── group_vars/
│   │       └── all.yml
│   └── projeto_c/
│       ├── hosts.yml
│       └── group_vars/
│           └── all.yml
│
├── playbooks/                  ← "QUE ORQUESTRAÇÃO rodar" (escolhe grupo + roles)
│   ├── test_trino.yml          ← chama a role trino_smoke
│   ├── test_opa.yml            ← (fase 2)
│   └── test_full_stack.yml     ← (fase 4)
│
└── .gitignore

```

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
# Fase 1 — Trino only
ansible-playbook -i inventory/projeto_madalena playbooks/test_trino.yml

# Quando quiser ligar a sonda OPA (sem mudar código):  Ligando a probe OPA sem editar nada.
ansible-playbook -i inventory/projeto_madalena playbooks/test_trino.yml -e smoke_opa_probe=true
```
