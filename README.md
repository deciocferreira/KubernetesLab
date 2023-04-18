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
Container é uma forma de realizar isolamento de recursos. Quando criamos um Pod podemos especificar a quantidade de CPU e Memória (RAM) que pode ser consumida em cada container. Quando algum container contém a configuração de limite de recursos o Scheduler fica responsável por alocar esse contêiner no melhor nó possível de acordo com os recursos disponíveis.

Podemos configurar dois tipos de recursos, CPU que é especificada em unidades de núcleos e Memória que é especificada em unidades de bytes. 

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

-> Gerenciamento de pods (deployments, statefulsets, daemonsets);

## Networking (Serviços, DNS, Ingress Controllers).

Primeira coisa que devemos entender é que o Kubernetes por padrão não fornece uma solução de networking de pods em nós diferentes, para que isso seja resolvido é necessário utilizar o que chamamos de pod networking. Para resolver esse problema foi criado o Container Network Interface. O CNI nada mais é do que um conjunto de plugins para resolver o problema de comunicação entre os pods. Há diversas soluções de pod networking como add-on, tais como: Calico, Weave-net e entre outros.

As características básicas da rede do k8s são:

- Todos os pods conseguem se comunicar entre eles em diferentes nodes;
- Todos os nodes podem se comunicar com todos os pods;
- Não utilizar NAT. 
 
O k8s organiza tudo dentro de namespaces. Nada mais é do que um cluster virtual dentro do próprio cluster físico do Kubernetes. Namespaces são uma maneira de dividir recursos de um cluster entre vários ambientes, equipes ou projetos. Por meio deles, podem ser realizadas limitações de segurança e de recursos dentro do cluster, tais como pods, replication controllers e diversos outros. Para visualizar os namespaces disponíveis no cluster, digite:
 
*kubectl get namespaces*
 
Dispositivos fora do cluster, por padrão, não conseguem acessar os pods criados, como é comum em outros sistemas de contêineres. Para expor um pod, execute o comando a seguir.

*kubectl expose pod nginx*
 
*Port Forwarding* **Caso queria encaminhar o tráfego da porta local para a porta do pod, podemos usar por exemplo: *kubectl port-forward nginx 8080:80***

*DNS* **Por padrão no K8s, os pods de DNS-core tem a capacidade de detectar os outros pods, dentro do seu próprio Namespace ou em outro.**

*Ingress* **Gerenciador de acesso externo aos serviços dentro do cluster. Ele atua como uma camada de abstração acima dos serviços e roteia as solicitações de entrada para os serviços apropriados com base nas regras definidas. Ele define um conjunto de regras de roteamento que determinam como o tráfego externo deve ser direcionado para os serviços dentro do cluster. Ele permite que você expõe vários serviços HTTP e HTTPS em um único endereço IP e porta, fornecendo assim um controle centralizado e flexível do tráfego. Podem ser implementados por meio de diferentes controladores, como o Nginx, Traefik, Istio, entre outros. Cada controlador pode oferecer diferentes recursos, como balanceamento de carga, redirecionamento, autenticação, criptografia, e etc.**

HPA - *Horizontal Pod AutoScaling*

Quando determinada métrica de aplicação alcançar determinado nível podemos configurar HPA para escalar um número de Pods necessários para que não haja gargalos na aplicação.
 
EndPoint - Sempre que criamos um service, automaticamente é criado um endpoint. O endpoint nada mais é do que o IP do pod que o service irá utilizar. Quando batemos nesse IP ele redireciona a conexão para o Pod através desse IP.

Para listar os EndPoints criados, execute o comando:

kubectl get endpoints 
 
## Componentes do K8s

- Master node
  - kube-apiserver é a central de operações do cluster k8s. Todas as chamadas, internas ou externas são tratadas por ele. Ele é o único que conecta no ETCD.

  - kube-scheduller usa um algoritmo para verificar em qual node o pod deverá ser hospedado. Ele verifica os recursos disponíveis do node para verificar qual o melhor node para receber aquele pod.
 
- Worker node 
  - kubelet interage com o Docker instalado no node e garante que os contêineres que precisavam estar em execução realmente estão.

  - kube-proxy é o responsável por gerenciar a rede para os contêineres, é o responsável por expor portas dos mesmos.
 
- Services: É uma forma de você expor a comunicação através de um *NodePort* ou *LoadBalancer* para distribuir as requisições entre diversos Pods daquele Deployment. Funciona como um balanceador de carga.

- Controllers: 
  - kube-controller-manager é o controle principal que interage com o kube-apiserver para determinar o seu estado. Se o estado não bate, o manager irá contactar o controller necessário para checar seu estado desejado. Tem diversos controllers em uso como: os endpoints, namespace e replication.
 
- Pods: É a menor unidade que você irá tratar no k8s. Pode conter mais de um contêiner e vale reassaltar que os diversos serviços sendo executados dessa maneira, compartilham o mesmo IP e demais recursos. Uma das boas razões para se ter mais de um contêiner é o fato de você ter os logs consolidados e centralizados.

Namespaces e quotas:
 - Supervisord é o responsável por monitorar e restabelecer, se necessário, o kubelet e o Docker. Por esse motivo, quando existe algum problema em relação ao kubelet, como por exemplo o uso do driver cgroup diferente do que está rodando no Docker, você perceberá que ele ficará tentando subir o kubelet frequentemente.
 
- Storage: No ETCD são armazenados o estado do cluster, rede e outras informações persistentes.

*Network e policies*
 
Principais Comandos
 
 <image src="https://user-images.githubusercontent.com/12403699/232628319-12e7f0a5-fb1b-4da1-99bf-155e77f212a4.png" width="700" height="350">

*kubectl run --image nginx --port 80* **Criação de Pod.**

*kubectl exec -ti podname --bash* **Acesso ao pod.**

*kubect delete pods* **Apaga determinado pod.**

**kubectl expose pods podname --type* **Expoe um tipo de node.**

*dry run* **Simulação de pods.**

*kubectl apply -f* **Aplica configurações de um arquivo .yaml**

## Referências
  
*https://kubernetes.io/blog/2020/05/21/wsl-docker-kubernetes-on-the-windows-desktop/*

*https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#install*
  
*https://k3d.io/v5.4.6/?h=install#other-installers*

*https://k3d.io/v5.4.6/usage/multiserver/*

*https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters*

*https://github.com/kubernetes/kubernetes/*

*https://kubernetes.io/docs/home/*
