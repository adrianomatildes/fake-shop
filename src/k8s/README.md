<!-- Criar o cluster -->
k3d cluster create dev \        
  --servers 1 \
  --agents 2 \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --k3s-arg "--disable=traefik@server:0"

<!-- Instalar o Service Nginx Controller -->
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/baremetal/deploy.yaml

<!-- Verificar se o svc está como LoadBalancer -->
kubectl get pods -n ingress-nginx -w

<!-- Caso não tiver altere -->
kubectl patch svc ingress-nginx-controller \
  -n ingress-nginx \
  -p '{"spec": {"type": "LoadBalancer"}}'

<!-- Manifestos do banco primeiro -->
cd k8s/db
k apply -f configmap.yaml -f secret.yaml -f service.yaml -f deployment.yaml

<!-- Manifestos da aplicação em segundo -->
cd ../dev
k apply -f configmap.yaml -f secret.yaml -f service.yaml -f deployment.yaml -f ingress.yaml