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
    └── camada3_tmp.yml          ← funcional em /tmp (file:// + memory catalog)

inventory/.../group_vars/pentaho.yml
docs/test_pentaho_scripts.md     ← relatório A–F + política de testes + fluxograma
```

- Testes SQL: modo PADRÃO read-only (`SELECT 1` + `system.metadata.catalogs`) — zero write, zero change.
  Modo ESTENDIDO (DDL/DML em RAM) disponível apenas se o catalog memory for habilitado em janela de change
  via `playbooks/enable_trino_memory_catalog.yml` (opcional, dormente).

## Arquitetura dos Testes
```
# HDFS vira sistema de arquivos local graças ao prefixo file://, então todos os scripts que operam sobre paths HDFS podem rodar 100% em sandbox — exceto os SQL, que precisam de um Trino real para validar a cadeia (TLS, auth, engine).

┌─ goodview (local) ───────────────────────────────┐
│  /tmp/smoke_<epoch>/                             │
│  ├── patched/     ← cópias dos scripts (sedeadas)│
│  ├── hdfs/        ← "HDFS fake" (FS local)       │
│  │   ├── dado_bruto/                             │
│  │   ├── rollback/                               │
│  │   └── tmp/                                    │
│  ├── dados/       ← shelldelete testa aqui       │
│  └── *.sql        ← queries que vão pra madalena │
└──────────────────────┬───────────────────────────┘
                       │ (só T5/T5b)
                       ▼
             ┌─ madalena (Trino real) ─┐
             │  system.catalogs        │  ← read-only
             │  memory.smoke_ansible   │  ← DDL/DML (opcional)
             └─────────────────────────┘
```

## Como rodar
```
# 1) Preparação
ansible-playbook -i inventory/projeto_madalena playbooks/test_pentaho_scripts.yml --syntax-check

# 2) Suite default — valida tudo sem tocar em nada
ansible-playbook -i inventory/projeto_madalena playbooks/test_pentaho_scripts.yml

```