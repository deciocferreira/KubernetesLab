# Instalação da aplicação Prometheus com K8s e Helm Charts

Em poucos passos, vamos ver como foi feita uma simples configuração para fazer o deploy do Prometheus.

Em um Cluster K8s, com o Helm devidamente configurado, vamos instalar o Prometheus. 

1. Visualizar qual a versão do chart disponível para instalação:

*helm search repo prometheus-community*

<image src="https://user-images.githubusercontent.com/12403699/233802717-d2d37efc-943c-495c-b1f8-f3b59299e16f.png" width="800" height="400">

2. Instalação do Prometheus via Chart:
  
*helm install meu-prometheus --version=14.8.0 prometheus-community/prometheus*

*helm list*

<image src="https://user-images.githubusercontent.com/12403699/233803097-56a02d3e-1d44-4602-b0f3-fa65cedfc80d.png" width="800" height="200"> 

3. Verificação dos recursos criados:
  
Após a ação poderemos checar que o Helm criou obejtos dentro do clulster, como os pods, deployments e services necessários para aplicação ser executada:

*kubectl get pods*

<image src="https://user-images.githubusercontent.com/12403699/233802753-a285594a-fad1-4f49-b6ff-6b4006dad735.png" width="800" height="200">
  
*kubectl get deploy*
  
<image src="https://user-images.githubusercontent.com/12403699/233802908-255c8379-9995-4a85-81fa-6264df119d0a.png" width="800" height="200"> 
  
*kubectl get replicaset* & *kubectl get svc*

<image src="https://user-images.githubusercontent.com/12403699/233802973-cb0f3b5d-8375-42b9-b70a-bbaadd593985.png" width="800" height="300"> 
  
*kubectl describe pod pod-aplicação*

<image src="https://user-images.githubusercontent.com/12403699/233802799-ca37e9dd-9ed5-44ef-8aa5-586f2226ecbf.png" width="800" height="400"> 

Podemos ver nas linhas Liveness e Readness que o Prometheus está sendo executado na porta 9090/TCP, com isso, faça um port Forward para acessar a aplicação:
  
<image src="https://user-images.githubusercontent.com/12403699/233802880-226455fd-904c-440f-bde6-eb2ec95bd590.png" width="800" height="100"> 
    
4. Agora acesse o navegador no endereço http://localhost:9091. 

<image src="https://user-images.githubusercontent.com/12403699/233802384-83bdb138-fc8f-4fa8-becd-05aa9ce8a854.png" width="1000" height="550"> 
  
Pronto, aplicação no ar localmente!  
