# KubernetesLab 

<image src="https://user-images.githubusercontent.com/12403699/227604690-54fb4263-a38a-4cd5-a4dc-951b19861625.png" width="80" height="80">

Orquestrador de containers. Responsável por organizar, controlar e gerenciar containers.

## Arquitetura

É um modelo baseado em control plane/workers, que constituem um cluster, onde para seu funcionamento é recomendado no mínimo três nós: o nó control-plane, responsável (por padrão) pelo gerenciamento do cluster, e os demais como workers, executores das aplicações.

<image src="https://user-images.githubusercontent.com/12403699/229541775-1db96bc0-641b-41f5-bc9e-a51f7c7072a7.png" width="800" height="500">

*ControlPlane* **Controla o Cluster. Responsável por garantir a saúde, capacidade e estado do Cluster.**
Componentes:
 - ETCD (função de banco do Cluster, guardando todo o estado do mesmo)
 - KubeAPIServer (Conversa com o ETCD e todo o cluster)
 - KubeScheduler (Responsável por fazer o agendamento e gerenciamento de novos containers: pods, volumes e etc)
 - Kube/CloudControllerManager (Responsável por garantir o estado de funcionamento do cluster, gerente dos outros controllers)

*Workers* **Contem as aplicações em execução.**
Componentes:
 - Kubelet (Agent do Kubernets dentro do node conversando com o KubeAPIServer)
 - KubeProxy (Faz a comunicação dos Pods com o restante da internet, expondo-o)
 
## Portas de comunicação:

- *KubeAPIServer:* **6443 TCP**

- *ETCD:* **2379/2380 TCP**

- *Kubelet:* **10250 TCP**

- *KubeScheduler:* **10251 TCP**

- *Kube/CloudControllerManager:* **10252 TCP**

- *NodePort* **30000/32767 TCP**

- *Weave Network* **6783/6784 TCP/UDP**

## Container
Container é uma forma de realizar isolamento de recursos.

Container Engine
- Responsável pela criação do container (docker, podman) pontos de montagem, rede,volume e verificação da saúde do container. Não conversa com o Kernel diretamente e depende do Container runtime para essa finalidade.

Container Runtime
- Especializado em conversar com o Kernel e executar os container, garantir que funcionem os isolamentos
Low Level executados pelo Kernel(Sandbox, RunC). 
High level executados por um Container Engine (ContainerD)

## OCI
- Open Container Initiative. Padronização para adoção de containers.

## Conceitos importantes:
*Pod* **Menor unidade de um CLuster e contém um ou mais containers. Tem somente um isolamento (IP, recurso e etc) e compartilha as informações com os devidos containers. Eles ficam dentro do Namespace kube-system.** 

*Deployment* **Responsável por criar infra de acordo com necessidade.**

*ReplicaSet* **Controlador de Pods que garante a quantidade e mantem replicas do Deployment.**

*Service* **Responsável por fazer com que os Pods fiquem acessível fora do Cluster ou Node. Expoe a aplicação para ser acessada.**

*Ingress* **Gerenciador de acesso externo aos serviços dentro do cluster. Ele atua como uma camada de abstração acima dos serviços e roteia as solicitações de entrada para os serviços apropriados com base nas regras definidas. Ele define um conjunto de regras de roteamento que determinam como o tráfego externo deve ser direcionado para os serviços dentro do cluster. Ele permite que você expõe vários serviços HTTP e HTTPS em um único endereço IP e porta, fornecendo assim um controle centralizado e flexível do tráfego. Podem ser implementados por meio de diferentes controladores, como o Nginx, Traefik, Istio, entre outros. Cada controlador pode oferecer diferentes recursos, como balanceamento de carga, redirecionamento, autenticação, criptografia, e etc.**

-> Gerenciamento de pods (deployments, statefulsets, daemonsets);

-> Elasticidade (HPA, VPA);

HPA - *Horizontal Pod AutoScaling*

Quando determinada métrica de aplicação alcançar determinado nível podemos configurar HPA para escalar um número de Pods necessários para que não haja gargalos na aplicação.

## Networking (Serviços, DNS, Ingress, Ingress Controllers).

Primeira coisa que devemos entender é que o Kubernetes não resolve como funciona a comunicação de pods em nós diferentes, para que isso seja resolvido é necessário utilizar o que chamamos de pod networking.

Ou seja, o k8s por padrão não fornece uma solução de networking out-of-the-box.

Para resolver esse problema foi criado o Container Network Interface, o CNI. O CNI nada mais é do que uma especificação e um conjunto de bibliotecas para a criação de soluções de pod networking, ou seja, plugins para resolver o problema de comunicação entre os pods.

Temos diversas solução de pod networking como add-on, cada qual com funcionalidades diferentes, tais como: Flannel, Calico, Romana, Weave-net, entre outros.

É importante saber as caracteristicas de cada solução e como elas resolvem a comunicação entre os pods.

Por exemplo, temos soluções que utilizam eBPF como é o caso do Cilium, ou ainda soluções que atuam na camada 3 ou na camada 7 do modelo de referencia OSI.

Dito isso, a melhor coisa é você ler os detalhes de cada solução e entender qual a melhor antende suas necessidades.

Eu gosto muito da Weave-net e será ela que iremos abordar durante o treinamento, na dúvida de qual usar, vá de Weave-net! :)

Para instalar o Weave-net execute o seguinte comando no nó control plane.
 
 O k8s organiza tudo dentro de namespaces. Por meio deles, podem ser realizadas limitações de segurança e de recursos dentro do cluster, tais como pods, replication controllers e diversos outros. Para visualizar os namespaces disponíveis no cluster, digite:
 
 Dispositivos fora do cluster, por padrão, não conseguem acessar os pods criados, como é comum em outros sistemas de contêineres. Para expor um pod, execute o comando a seguir.

kubectl expose pod nginx
Será apresentada a seguinte mensagem de erro:

error: couldn't find port via --port flag or introspection
See 'kubectl expose -h' for help and examples
O erro ocorre devido ao fato do k8s não saber qual é a porta de destino do contêiner que deve ser exposta (no caso, a 80/TCP). Para configurá-la, vamos primeiramente remover o nosso pod antigo

## Comandos e parâmetros:

*kubectl run --image nginx --port 80* **Criação de Pod.**

*kubectl exec -ti podname --bash* **Acesso ao pod.**

*kubect delete pods* **Apaga determinado pod.**

**kubectl expose pods podname --type* **Expoe um tipo de node.**

*dry run* **Simulação de pods.**

*kubectl apply -f* **Aplica configurações de um arquivo .yaml**


## Referências 

*https://k3d.io/v5.4.6/?h=install#other-installers*

*https://k3d.io/v5.4.6/usage/multiserver/*

*https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters*

*https://github.com/kubernetes/kubernetes/*

*https://kubernetes.io/docs/home/*
