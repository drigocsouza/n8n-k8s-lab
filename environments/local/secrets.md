# Secrets (local)

Nenhum Secret vai pro git — todos criados via `kubectl create secret` ou pelo
próprio Helm chart, nunca escritos em arquivo versionado. O `.env.local`
guarda uma cópia de apoio fora do git (listado no .gitignore) só dos valores
que nós geramos manualmente.

## Secrets no namespace n8n

| Secret               | Chaves                                                        | Criado por                              |
|-----------------------|-----------------------------------------------------------------|-------------------------------------------|
| `n8n-db-secret`        | `password`                                                       | `kubectl create secret generic` (manual)   |
| `n8n-core-secrets`     | `N8N_ENCRYPTION_KEY`, `N8N_HOST`, `N8N_PORT`, `N8N_PROTOCOL`     | `kubectl create secret generic` (manual)   |
| `n8n-task-runners`     | `N8N_RUNNERS_AUTH_TOKEN`                                         | Chart Helm, automático                     |
| `sh.helm.release.v1.*` | (release history do Helm, não é segredo de aplicação)           | Helm, interno                              |

Os dois primeiros são referenciados no `n8n-values.yaml` via
`database.passwordSecret` e `secretRefs.existingSecret`. O `n8n-task-runners`
é criado e gerenciado pelo próprio chart — token do broker local (porta 5679)
usado por main/worker para executar código de nodes (ex: "Code") de forma
isolada, existe por padrão mesmo sem `taskRunners.external` configurado.

## N8N_ENCRYPTION_KEY — cuidado especial

Gerada uma vez com `openssl rand -hex 16` e guardada em `.env.local` (fora do
git). **Nunca regenerar** — ela criptografa as credenciais salvas dentro do
n8n (conexões, tokens de outras integrações). Se for perdida ou trocada, um
restore do banco Postgres devolve essas credenciais ilegíveis, mesmo com o
banco intacto. Mesmo princípio citado em um padrão de arquitetura enterprise de referência para
produção — aqui vale igual, só que sem Key Vault por trás.

## O que muda na Fase 2 (Azure)

Os valores que hoje moramos manualmente (senha do Postgres, redis,
encryption key) saem do Kubernetes Secret puro e passam a viver no Key
Vault, lidos pelos pods via Secrets Store CSI Driver + Workload Identity —
sem senha própria armazenada em lugar nenhum do cluster. O
`n8n-task-runners`, por ser auto-gerado pelo chart, provavelmente continua
igual — não é algo que precisamos gerenciar manualmente nem lá.

## Pegadinha: N8N_PORT != porta pública

`N8N_PORT` é a porta em que o processo n8n escuta DENTRO do container, não a
porta pública de acesso. Setar `N8N_PORT=443` faz o processo tentar escutar
em 443, mas o Service do chart aponta pra 5678 — o pod nunca fica "Ready" e
entra em restart loop. Manter sempre `N8N_PORT=5678` (o padrão), mesmo
quando o acesso externo é por 443 via Ingress/Application Gateway — quem
faz TLS termination e expõe a porta pública é a camada de Ingress, não o
processo n8n.
