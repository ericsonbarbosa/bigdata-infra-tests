# bigdata-infra-tests-
Smoke test superficial e rápido que verifica se o sistema "liga" e responde minimamente após deploy/instalação.

## Versões

```
# Versão Ansible
ansible-core==2.13.13

# Versão Python
python3.8
```


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
│   │   ├── hosts.yml           ← ansible_user global + grupos trino/postgres/mongo
│   │   └── group_vars/
│   │       └── trino.yml       ← variáveis DO PROJETO (vars do Trino)
│   │       └── postgres.yml    ← pg_port, pg_service, pg_expected_listen_addresses...
│   │       └── mongo.yml       ← mongo_port, mongo_service, mongo_conf...
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


## Notas do ansible.cfg:
```
# Sem inventory fixo: com múltiplos projetos (projeto_madalena, futuros projeto_goodview…), o inventário vai na CLI — evita executar acidentalmente contra o host errado.

# stdout_callback = yaml exige a coleção community.general (vem no pacote ansible; se usar só ansible-core, remova a linha ou instale a coleção).

# ssh_args já cobre o -o StrictHostKeyChecking=no do seu comando SSH.

# [privilege_escalation] com become=True global: as tarefas de leitura que precisam de sudo (slurp do etc/) já funcionam sem repetir become tarefa a tarefa (o play também declara become: true, sem conflito).
```