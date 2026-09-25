# n8n-k8s-lab

Réplica de aprendizado da arquitetura de referência n8n na Azure (baseado em um padrão de referência enterprise para n8n em Kubernetes),
primeiro local (kind/WSL) e depois no AKS.

## Fase 1 — Local (kind)
- [x] kind + kubectl + helm instalados
- [x] cluster kind criado (clusters/kind-config.yaml)
- [x] ingress-nginx
- [x] Postgres + Redis in-cluster
- [x] Secrets (N8N_ENCRYPTION_KEY, db, redis, task-runners) - documentado em environments/local/secrets.md
- [x] helm install n8n (queue mode: main + worker + webhook-processor)
- [x] validar fila (webhook -> worker) - confirmado via logs: worker processou job 1 e job 2

## Fase 2 — Azure (AKS, subscription Azure pessoal — id fora do git)
Segue o doc de arquitetura de referência: AKS privado, PostgreSQL Flexible Server,
Azure Managed Redis, Key Vault + Workload Identity, AGIC/App Gateway WAF.
Ainda não iniciado.

## Chart usado
oci://ghcr.io/n8n-io/n8n-helm-chart/n8n (oficial n8n-io/n8n-hosting)
