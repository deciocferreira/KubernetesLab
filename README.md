# Kubernetes 

<image src="https://user-images.githubusercontent.com/12403699/227604690-54fb4263-a38a-4cd5-a4dc-951b19861625.png" width="80" height="80">

Orquestrador de containers. Responsável por organizar, controlar e gerenciar containers.

## Container
Container é uma forma de realizar isolamento de recursos. Quando criamos um Pod podemos especificar a quantidade de CPU e Memória (RAM) que pode ser consumida em cada container. Quando algum container contém a configuração de limite de recursos o Scheduler fica responsável por alocar esse contêiner no melhor nó possível de acordo com os recursos disponíveis.

Podemos configurar dois tipos de recursos, CPU que é especificada em unidades de núcleos e Memória que é especificada em unidades de bytes. 

Container Engine
- Responsável pela criação do container (docker, podman) pontos de montagem, rede,volume e verificação da saúde do container. Não conversa com o Kernel diretamente e depende do Container runtime para essa finalidade.

Container Runtime
- Especializado em conversar com o Kernel e executar os container, garantir que funcionem os isolamentos
Low Level executados pelo Kernel(Sandbox, RunC). 
High level executados por um Container Engine (ContainerD)

OCI
- Open Container Initiative. Padronização para adoção de containers.
 
## Arquitetura do K8s

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
 
## Portas de comunicação

- *KubeAPIServer:* **6443 TCP**

- *ETCD:* **2379/2380 TCP**

- *Kubelet:* **10250 TCP**

- *KubeScheduler:* **10251 TCP**

- *Kube/CloudControllerManager:* **10252 TCP**

- *NodePort* **30000/32767 TCP**

- *Weave Network* **6783/6784 TCP/UDP**  
 
## Conceitos importantes e Componentes do K8s

- Master node
  - kube-apiserver é a central de operações do cluster k8s. Todas as chamadas, internas ou externas são tratadas por ele. Ele é o único que conecta no ETCD.

  - kube-scheduller usa um algoritmo para verificar em qual node o pod deverá ser hospedado. Ele verifica os recursos disponíveis do node para verificar qual o melhor     node para receber aquele pod.
 
- Worker node 
  - kubelet interage com o Docker instalado no node e garante que os contêineres que precisavam estar em execução realmente estão.

  - kube-proxy é o responsável por gerenciar a rede para os contêineres, é o responsável por expor portas dos mesmos.
 
- Services: É uma forma de você expor a comunicação através de um *NodePort* ou *LoadBalancer* para distribuir as requisições entre diversos Pods daquele Deployment. Funciona como um balanceador de carga.

- Controllers: 
  - kube-controller-manager é o controle principal que interage com o kube-apiserver para determinar o seu estado. Se o estado não bate, o manager irá contactar o controller necessário para checar seu estado desejado. Tem diversos controllers em uso como: os endpoints, namespace e replication.
 
- Pods: É a menor unidade que você irá tratar no k8s dentro de um cluster. Pode conter mais de um contêiner e vale reassaltar que os diversos serviços sendo executados dessa maneira, compartilham o mesmo IP e demais recursos. Uma das boas razões para se ter mais de um contêiner é o fato de você ter os logs consolidados e centralizados.

*Deployment* - É um recurso com a responsabilidade de instruir o Kubernetes a criar, atualizar e monitorar a saúde das instâncias de suas aplicações. É responsável por gerenciar o seu ReplicaSet, ou seja, o Deployment é quem vai determinar a configuração de sua aplicação e como ela será implementada. O Deployment é o controller que irá cuidar, por exemplo, uma instância de sua aplicação por algum motivo for interrompida. O Deployment controller irá identificar o problema com a instância e irá criar uma nova em seu lugar. **Cria infra de acordo com necessidade.**

*ReplicaSet* Garante a quantidade solicitada de pods e os recursos necessários para um Deployment. Uma vez que o Deployment é criado, é o ReplicaSet que controla a quantidade de pods em execução, caso algum pod seja finalizado, ele que irá detectar e solicitar que outro pod seja executado em seu lugar, garantindo assim a quantidade de réplicas solicitadas. Age como um controlador que garante e mantem as replicas do Deployment.
 
<image src="https://user-images.githubusercontent.com/12403699/233199822-2a4a65a2-1f1e-4d45-9289-9788c01949f6.png" width="800" height="400"> 

*Service* **Responsável por fazer com que os Pods fiquem acessível fora do Cluster ou Node. Expoe a aplicação para ser acessada.**

## Gerenciamento de pods (statefulsets)

*DaemonSet* é semelhante ao ReplicaSet, com a diferença que quando você utiliza o DaemonSet você não especifica o número de réplicas, ele subirá um pod por node em seu cluster. Ele ótimo para serviços que necessitem rodar em todos os nodes do cluster, como por exemplo, coletores de logs e agentes de monitoração.

<image src="https://user-images.githubusercontent.com/12403699/233205526-1826b85b-696d-4321-99f8-baf1bbe331a0.png" width="800" height="400"> 
 
## Rollouts e Rollbacks

Usamos quando precisamos verificar quais foram as modificações que aconteceram em seu Deployment ou DaemonSet, como se fosse um versionamento. 

<image src="https://user-images.githubusercontent.com/12403699/233206212-22965bef-0d9e-4fef-a089-c0f50adecc48.png" width="800" height="400"> 
 
Por padrão, o DaemonSet guarda apenas as 10 últimas revisions. Para alterar a quantidade máxima de revisions no nosso Daemonset, execute o seguinte comando.

kubectl edit daemonsets.apps daemon-set-primeiro 

Para atualizar pods do DaemonSet precisamos recriá-lo ou destruir todos os pods relacionado a ele. Como boa prática, temos a opção RollingUpdate
 
## Networking (Ingress Controllers).

Primeira coisa que devemos entender é que o Kubernetes por padrão não fornece uma solução de networking de pods em nós diferentes, para que isso seja resolvido é necessário utilizar o que chamamos de pod networking. Para resolver esse problema foi criado o Container Network Interface. O CNI nada mais é do que um conjunto de plugins para resolver o problema de comunicação entre os pods. Há diversas soluções de pod networking como add-on, tais como: Calico, Weave-net e entre outros.

As características básicas da rede do k8s são:

- Todos os pods conseguem se comunicar entre eles em diferentes nodes;
- Todos os nodes podem se comunicar com todos os pods;
- Não utilizar NAT. 
 
O k8s organiza tudo dentro de namespaces. Nada mais é do que um cluster virtual dentro do próprio cluster físico do Kubernetes. Namespaces são uma maneira de dividir recursos de um cluster entre vários ambientes, equipes ou projetos. Por meio deles, podem ser realizadas limitações de segurança e de recursos dentro do cluster, tais como pods, replication controllers e diversos outros. Para visualizar os namespaces disponíveis no cluster, digite:
 
*kubectl get namespaces*
 
Como visto anteriormente, temos a possibilidade de adicionar limites de memória e cpu para cada container que subir nesse Namespace, se algum container for criado dentro do Namespace sem as configurações de Limitrange, o container irá herdar as configurações de limites de recursos do Namespace.
 
<image src="https://user-images.githubusercontent.com/12403699/232863578-c219bc4a-b43f-4aeb-b927-3ecafa615d68.png" width="800" height="400">
  
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
 
Namespaces e quotas:
 - Supervisord é o responsável por monitorar e restabelecer, se necessário, o kubelet e o Docker. Por esse motivo, quando existe algum problema em relação ao kubelet, como por exemplo o uso do driver cgroup diferente do que está rodando no Docker, você perceberá que ele ficará tentando subir o kubelet frequentemente.
 
- Storage: No ETCD são armazenados o estado do cluster, rede e outras informações persistentes.

*Network e policies*
 
##Kubectl Taint
 
*Taint* no K8s é adicionar propriedades ao nó do cluster para impedir que os pods sejam alocados em nós inapropriados. Por exemplo, todo nó master do cluster é marcado para não receber pods que não sejam de gerenciamento do cluster. O nó master está marcado com o taint NoSchedule, assim o scheduler do Kubernetes não aloca pods no nó master, e procura outros nós no cluster sem essa marca.

<image src="https://user-images.githubusercontent.com/12403699/232866300-a86a6a3c-bcf5-41e8-8b0a-5afab349be53.png" width="800" height="500"> 
 
## Labels
 
São utilizadas para a organização do cluster, vamos listar pods que tenha a marcação "Warsaw" por exemplo:
 
<image src="https://user-images.githubusercontent.com/12403699/233195389-d9b32da6-bd7e-4656-8ffe-689802db6378.png" width="800" height="500">   

## Node Selector 

É uma forma de classificar nossos nodes, através das labels, como por exemplo se é produção ou não, se consome muita CPU ou muita RAM, se precisa estar em determinado Namespace.
 
## Kubectl edit

Se usa quando é necessário editar algum deployment ou qualquer objeto Kubernetes(pods, services, configmaps, e etc). O comando abre o arquivo de configuração YAML do objeto em um editor de texto para que você possa editá-lo diretamente. É importante lembrar que a edição direta pode ser perigosa se você não tiver cuidado para não alterar configurações importantes. Sempre faça um backup da configuração antes de editá-la e verifique as alterações antes de aplicá-las.

*kubectl edit deployment deployment.yaml* e adicionar informações de *Label* e ou *nodeSelector* conforme necessidade.
  
Principais Comandos
 
 <image src="https://user-images.githubusercontent.com/12403699/232628319-12e7f0a5-fb1b-4da1-99bf-155e77f212a4.png" width="700" height="350">

*kubectl run --image nginx --port 80* **Criação de Pod.**

*kubectl exec -ti podname --bash* **Acesso ao pod.**

*kubect delete pods* **Apaga determinado pod.**

**kubectl expose pods podname --type* **Expoe um tipo de node.**

*dry run* **Simulação de pods.**

*kubectl apply -f* **Aplica configurações de um arquivo .yaml**

## Referências

*https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#clean-up-policy*
  
*https://kubernetes.io/blog/2020/05/21/wsl-docker-kubernetes-on-the-windows-desktop/*

*https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#install*
  
*https://k3d.io/v5.4.6/?h=install#other-installers*

*https://k3d.io/v5.4.6/usage/multiserver/*

*https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal-clusters*

*https://github.com/kubernetes/kubernetes/*

*https://kubernetes.io/docs/home/*
