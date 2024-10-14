# Instalação Rancher - Nginx - CertManager - ClusterIssuer

## Instalação do Nginx Controller
~~~bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm search repo ingress-nginx/ingress-nginx --versions
helm show values ingress-nginx/ingress-nginx --version 4.11.2 > values-ingress-nginx.yaml
helm upgrade -i ingress-nginx ingress-nginx/ingress-nginx \
--namespace nginx \
--create-namespace \
--values values-ingress-nginx.yaml \
--version=4.11.2
~~~
**Explicação:**
- **helm repo add ingress-nginx**: Adiciona o repositório oficial do `ingress-nginx`, que é o controlador de Ingress NGINX usado para gerenciar o tráfego de entrada no Kubernetes.
- **helm repo update**: Atualiza os repositórios do Helm para garantir que as versões mais recentes dos pacotes estejam disponíveis.
- **helm search repo ingress-nginx**: Lista as versões do repositorio do `ingress-nginx`
- **helm show values ingress-nginx/ingress-nginx --version 4.11.2 > values-ingress-nginx.yaml**: Copia os valores de configuração de uma versão especifica do ingress-nginx para um arquivo especifico.
- **helm upgrade -i ingress-nginx**: Instala ou atualiza o controlador Ingress NGINX no cluster Kubernetes. O nome do release é `ingress-nginx` e o gráfico Helm é baixado do repositório `ingress-nginx/ingress-nginx`.
- **--namespace nginx**: Define o namespace onde o controlador será instalado.
- **--create-namespace**: Cria o namespace `nginx` se ele ainda não existir.
- **--values values-default.yaml**: Usa o arquivo de configuração `values-default.yaml` para personalizar a instalação.
- **--version 4.11.2**: Especifica a versão exata do controlador NGINX que será instalada.

## Instalação do Cert-Manager
~~~bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm search repo jetstack/cert-manager --versions
helm upgrade -i cert-manager jetstack/cert-manager \
--namespace cert-manager \
--version v1.16.1 \
--set crds.enabled=true \
--create-namespace
~~~
**Explicação:**
- **helm repo add jetstack**: Adiciona o repositório do `cert-manager`, um operador que gerencia certificados SSL/TLS no Kubernetes.
- **helm repo update**: Atualiza os repositórios do Helm.
- **helm search repo jetstack**: Lista as versões do repositório do `cert-manager`
- **helm upgrade -i cert-manager jetstack/cert-manager**: Instala ou atualiza o `cert-manager` no cluster Kubernetes.
- **--namespace cert-manager**: Define o namespace onde o `cert-manager` será instalado.
- **--version v1.16.1**: Especifica a versão exata do `cert-manager` que será instalada.
- **--set crds.enabled=true**: Ativa a instalação dos Custom Resource Definitions (CRDs) que são necessários para o funcionamento do `cert-manager`.
- **--create-namespace**: Cria o namespace `cert-manager` caso ele não exista.

## Aplicar Deployment e Consultar ClusterIssuer
~~~bash
kubectl apply -f letsencrypt-clusterissuer.yaml
kubectl get clusterissuer
~~~
**Explicação:**
- **kubectl apply -f letsencrypt-clusterissuer.yaml**: Aplica o manifesto YAML, criando o `ClusterIssuer` no cluster.
- **kubectl get clusterissuer**: Verifica se o `ClusterIssuer` foi criado com sucesso, listando os recursos do tipo `ClusterIssuer` no cluster.

## Instalação do Rancher
~~~bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update rancher-stable
helm search repo rancher-stable/rancher --versions
helm upgrade -i rancher rancher-stable/rancher \
--namespace cattle-system \
--create-namespace \
--set hostname=rancher.daniel-lab.dominio.io \
--set replicas=2 \
--set ingress.tls.source=secret \
--version v2.9.2 \
--set ingress.extraAnnotations.'cert-manager\.io/cluster-issuer'=letsencrypt-prod \
--set ingress.ingressClassName=nginx 
~~~
**Explicação:**
- **helm repo add rancher-stable**: Adiciona o repositório Helm oficial do Rancher.
- **helm repo update rancher-stable**: Atualiza o repositório para garantir que você tenha a versão mais recente do Rancher disponível.
- **helm search repo rancher-stable**: Lista as versões do repositório do `rancher-stable`
- **helm upgrade -i rancher rancher-stable/rancher**: Instala ou atualiza o Rancher no cluster.
- **--namespace cattle-system**: Instala o Rancher no namespace `cattle-system`.
- **--create-namespace**: Cria o namespace `cattle-system` caso ainda não exista.
- **--set hostname=rancher.daniel-lab.o2b.io**: Define o hostname onde o Rancher será acessado (substitua por seu próprio domínio).
- **--set replicas=2**: Configura o Rancher para rodar com 2 réplicas (alta disponibilidade).
- **--set ingress.tls.source=secret**: Configura o Rancher para usar um segredo Kubernetes como fonte de TLS.
- **--version v2.9.2**: Especifica a versão do Rancher a ser instalada.
- **--set ingress.extraAnnotations.'cert-manager\.io/cluster-issuer'=letsencrypt-prod**: Anota o Ingress do Rancher para usar o `ClusterIssuer` `letsencrypt-prod`, emitindo certificados TLS via Let's Encrypt.
- **--set ingress.ingressClassName=nginx**: Define que o Ingress NGINX será usado para roteamento de tráfego.

**Em casos de erros como `unable to read CA file from /etc/kubernetes/ssl/certs/serverca: open /etc/kubernetes/ssl/certs/serverca: no such file or directory` na adição de um novo cluster ao rancher, deverá adicionar ao comando o seguinte valor `--set agentTLSMode=system-store` Pois com ele irá configura o modo TLS do agente para `system-store`, que usa certificados do sistema.**