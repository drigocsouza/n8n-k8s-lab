# Ingress NGINX (local)

Instalado via manifest oficial do projeto kind (provider "kind"):
https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

Motivo de usar o manifest "kind" e não o Helm chart genérico do ingress-nginx:
ele já vem com o `hostPort` configurado pra funcionar com o port mapping 80/443
que definimos em clusters/kind-config.yaml - sem isso, o LoadBalancer do Service
fica "Pending" para sempre (kind não tem cloud controller pra prover IP externo).

Na Azure isso vira Application Gateway + AGIC, que lê os mesmos objetos `Ingress`
e escreve as regras de roteamento automaticamente.
