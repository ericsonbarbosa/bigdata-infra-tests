# Automatização de testes em pipiline de Big Data
Smoke test superficial e rápido que verifica se o sistema "liga" e responde minimamente após deploy/instalação.

## Schema de pastas

```
playbooks/test_pentaho_scripts.yml
roles/pentaho_smoke_scripts/
├── defaults/main.yml            ← lista dos 11 scripts
└── tasks/
    ├── main.yml                 ← importa os blocos abaixo
    ├── camada0_prereqs.yml      ← .env, certs, binários, bug A (WARN)
    ├── camada1_contrato.yml     ← stat + bash -n dos 11
    ├── camada2_comportamento.yml← caminhos de falha sem mutação
    ├── camada3_tmp.yml          ← funcional em /tmp (file:// + memory catalog)
    ├── camada3_sandbox.yml      ← integração HDFS opcional (tag integracao_hdfs)
    └── canarios.yml             ← bugs B–F como WARN vivo
inventory/.../group_vars/pentaho.yml
docs/test_pentaho_scripts.md     ← relatório A–F + política de testes + fluxograma
```

- Testes SQL: modo PADRÃO read-only (`SELECT 1` + `system.metadata.catalogs`) — zero write, zero change.
  Modo ESTENDIDO (DDL/DML em RAM) disponível apenas se o catalog memory for habilitado em janela de change
  via `playbooks/enable_trino_memory_catalog.yml` (opcional, dormente).

## Como rodar
```
# 1) Preparação
ansible-playbook -i inventory/projeto_madalena playbooks/test_pentaho_scripts.yml --syntax-check

# 2) Suite default (read-only) — valida tudo sem tocar em nada
ansible-playbook -i inventory/projeto_madalena playbooks/test_pentaho_scripts.yml

# 3) Janela de change (1x): habilita catalog memory no Trino (one-shot do admin)
ansible-playbook -i inventory/projeto_madalena playbooks/enable_trino_memory_catalog.yml

# 4) Suite estendida — com memory ativo, T5+ testa DDL/DML em RAM (detecta sozinho)
ansible-playbook -i inventory/projeto_madalena playbooks/test_pentaho_scripts.yml

# 5) Opcional: completa com timeout do pipelinemove (+60s)
ansible-playbook -i inventory/projeto_madalena playbooks/test_pentaho_scripts.yml -e smoke_timeout_test=true

```