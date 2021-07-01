# AULA 2 - DESVENDANDO O KUBERNETES

### Kubernetes

* **Problemas do Docker solucionados**

  * Orquestração de containers

  * Automatização de processos

* **Vantagens**

  * Resiliência e escalabilidades nas aplicações (evitar queda da aplicação)

  * Balanceamento de cargas entre as réplicas, no momento de escalagem da aplicação (evitar a sobrecarga em uma das réplicas)

  * _Service discovery_ = cataloga os endereços do local onde cada processo foi enviado 

  * Recurso de _self healing_ = verificação do status dos containers (reinício do container em caso de problema)

  * Existência de estratégias de atualizações para evitar _downtime_ (fora do ar)

* **Onde usá-lo?**

  * Aplicações que exigem alta disponibilidade (**não podem cair**)

    * **Exemplo**: internet banking, e-commerce, gateway de pagamento

  * Microsserviços

  * Aplicações monolitos centradas em uma única ferramenta

* **Estrutura**

  * **Definição**

    * Formado por 1 _cluster_ (conjunto de máquinas), nos quais cada máquina exerce um panel (**Control Plane** ou **Node**)
    
    * **Node**: responsável pela execução dos containers das aplicações

    * **Control Plane**: gerência os **nodes** e orquestra todo o _cluster_

  * **Conceitos**

    * **Kubernetes Control Plane** (gerência o _cluster_ Kubernetes)

      * **Kube API Server** = responsável pela comunicação do _cluster_ do o "client" (CLI ou outro Kube API Server)

      * **ETCD** (banco de dados "chave-valor") =  armazenamento de dados do Kubernetes

        > OBS: a coleta dos dados **nunca** é feita diretamente com o ETCD. O Kube API Server tem a função de fazer a comunicação entre "DB" e "Client" 

      * **Kube Scheduler** = responsável pela alocação e execução de cada processo

      * **Kube Controller Manager** = execução e gerenciamento dos controladores do Kubernetes (tomada de decisões)

    * **Kubernetes Node**

      * **Kubelet** = monitoramento (reportes de status) e execução dos containers no **node**

      * **Kube Proxy** = responsável pelas comunicações de rede com o _cluster_

      * Container Runtime Interface (**CRI**) = armazenamentos de especificações necessárias para ser utilizado dentro do Kubernetes

  * **Docker descontinuado como CRI no Kubernetes 1.20** (Motivo)

    Docker utilizado o **Dockershim** como adaptador entre a interface do **CRI** com a do **Docker**

    * **Solução** = utilizar outras tecnologias (_Container-d_ ou _Cri-o_) ao utilizar o Kubernetes

### k3d

> **k3d** = versão do k3s para rodar em containers, ou seja, cada **node** do _cluster_ é um container (+ rápido e leve)

* **Instalação do k3d**: `$ curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash`

* **Instalação do Kubectl**: 

  * `$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"`

  * `$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl`

  * **Configuração do Zsh (autocomplete)**

    * `$ code ~/.zshrc`

    * Adicionar: `source <(kubectl completion zsh)`

* **Comandos**

  * **Criação de um _cluster_ Kubernetes com 1 node**

    * Sintaxe: `$ k3d cluster create <nome>`

    * Exemplo: `$ k3d cluster create meuprimeirocluster`

  * **Visualizar todos os nodes do _cluster_ Kubernetes**

    * Sintaxe: `$ kubectl get nodes`

  * **Listagem dos containers utilizados para a criação do _cluster_ Kubernetes**

    * Sintaxe: `$ docker container ls -a`

  * **Criação de um _cluster_ Kubernetes sem _load balancer_**

    > Utilizar esse comando **apenas** em clusters com apenas **1 node**

    * Sintaxe: `$ k3d cluster create <nome> --no-lb`

    * Exemplo: `$ k3d cluster create meusegundocluster --no-lb`

  * **Listagem dos _clusters_  Kubernetes**

    * Sintaxe: `$ k3d cluster list`

      > **Servers** = _Control Planes_

      > **Agents** = **nodes**

  * **Listagem de Pod, Services, Deployment, ReplicaSet**: `$ kubectl get all`

  * **Remoção de um _cluster_ kubernetes**

    * Sintaxe: `$ k3d cluster delete <cluster>`

    * Exemplo: `$ k3d cluster delete meuprimeirocluster`

  * **Criação de um _cluster_ com x server e y agents**

    > Lembrando: **server** = _control plane_ | **agents** = _node_

    * Sintaxe: `$ k3d cluster create <nome> --servers <número> --agents <número>`

    * Exemplo: `$ k3d cluster create meucluster --servers 3 --agents 3` 

  * Iniciar um _cluster_ Kubernetes

    * Sintaxe: `$ k3d cluster start <nome>`

    * Exemplo: `$ k3d cluster start meucluster`

  * Parar um _cluster_ Kubernetes

    * Sintaxe: `$ k3d cluster stop <nome>`

    * Exemplo: `$ k3d cluster stop meucluster`

### Deploy de um _cluster_ Kubernetes

* **Conceito: 3 objetos do _cluster_ Kubernetes**

  * **Pod**

    * **Definição**: menor objeto do _cluster_

    * **Função**: execução de containers (1 ou + ao mesmo tempo)

      > Esses containers podem dividem o mesmo IP e mesmo sistema de arquivo do **Pod** (compartilhamento de recursos)

    * Colocar todos os container de um aplicação em 1 **Pod**? (**NÃO PODE FAZER ISSO**)

      * Na criação de réplicas de um _cluster_ Kubernetes, há a **criação de réplicas do Pod** e não de container

      * Se colocar todos os container de uma aplicação em um mesmo **Pod**, na replicação, toda a aplicação será replicado . Com isso há **perdas na escalabilidade**.

    * **Sidecar pattern**: [medium](https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-sidecar-container-pattern-6d8c21f873d)

    * **Limitações**

      * Não tem a capacidade de fornecer resiliência e escalabilidade, por ser o menos objeto do _cluster_

        > O **Pod** ao ser removido, ele não é recriado (não tem resiliência)

      * Não é possível criar réplicas de Pod através do arquivo manifesto

  * **ReplicaSet**

    * **Definição**: gerenciar a escalabilidade e resiliência dos **Pods**

    * **Função**: controlar, de forma declarativa, o número de réplicas de um determinado **Pod** no _cluster_ Kubernetes

  * **Deployment**

    * **Definição**: gerenciador de ReplicaSets

    * **Função**: toda vez que ocorrer uma alteração no **template** de um **Pod** (alteração na versão ou no comportamento), o **Deployment** recriará um novo ReplicaSet (com os novos **templates**)

      > A troca entre as versões de um ReplicaSet é feita de forma progressiva

    * **Vantagens**

      * Escalabilidade

      * Resiliência

      * Troca de versão automatizada (**garantia de downtime 0**)

### Arquivo de manifesto de um **Pod**

* **Definição**
  
  * Sempre necessário quando for utilizar o Kubernetes

  * Arquivo em formato YAML

* **Função**: definir todas as configurações de um objeto

* **Campos obrigatórios na criação de um manifesto**

  * **apiVersion**
  
    * Função: definir o agrupamento de API, ou seja, a separação de alfa, beta, pertence a um recurso específico ou recurso final
    
    * Sintaxe: `apiVersion: valor`

    * Exemplo: `apiVersion: v1`

    > Qual apiVersion utilizar? `$ kubectl api-resources`

  * **kind**

    * Função: definir o objeto a ser declarado

    * Sintaxe: `kind: valor`

    * Exemplo: `kind: Pod`

  * **metadata**

    * Função: definir os metadados do objeto

    * Definir o nome do objeto

      * Sintaxe 

        ```yml
        metadata:
          name: valor
        ```

      * Exemplo

        ```yml
        metadata:
          name: meupod
        ```

  * **spec**

    * Função: definir a especificação do objeto

    * Definir o(s) container(s) à serem executados

      * Sintaxe

        ```yml
        spec:
          containers:
            # configurações do(s) container(s)
        ```

      * Exemplo

        ```yml
        spec:
          containers:
            - name: site
              image: kubedevio/nginx-color:blue
              ports:
                - containerPort: 80
        ```

* **Comandos**

  * **Criar o objeto no _cluster_ Kubernetes**

    * Sintaxe: `$ kubectl create -f <arquivo manifesto>`

    * Exemplo: `$ kubectl create -f ./meupod.yml`

  * **Remover o objeto do _cluster_ Kubernetes**

    * Sintaxe

      * `$ kubectl delete -f <arquivo manifesto>`

      * `$ kubectl delete pod <nome>`

    * Exemplo: `$ kubectl delete -f meupod.yml`

  * **Visualizar o objeto no _cluster_ Kubernetes**

    * Sintaxe: `$ kubectl get pods`

  * **Visualizar mais detalhes do objeto no _cluster_ Kubernetes**

    * Sintaxe: `$ kubectl describe pod <arquivo manifesto>`

    * Exemplo> `$ kubectl describe pod meupod`

  * **Binding da porta da maquina local com a porta do Pod (container em execução no Pod)** 

    * Sintaxe: `$kubectl port-forward <pod>/<nome do pod> <porta maquina local>:<porta Pod>`

    * Exemplo: `$ kubectl port-forward pod/meupod 8080:80`

  * Listagem dos pods com um output mais detalhado (**wide**): `$ kubectl get pods -o wide`

### Arquivo de manifesto de um **ReplicaSet**

* **Campos obrigatórios na criação de um manifesto**

  * **apiVersion**
    
    * Função: definir o agrupamento de API, ou seja, a separação de alfa, beta, pertence a um recurso específico ou recurso final
    
    * Sintaxe: `apiVersion: app/valor`

    * Exemplo: `apiVersion: apps/v1`

    > Qual apiVersion utilizar? `$ kubectl api-resources`

  * **kind**

    * Função: definir o objeto a ser declarado

    * Sintaxe: `kind: valor`

    * Exemplo: `kind: ReplicaSet`

  * **metadata**

    * Função: definir os metadados do objeto

    * Definir o nome do objeto

      * Sintaxe 

        ```yml
        metadata:
          name: valor
        ```

      * Exemplo

        ```yml
        metadata:
          name: meureplicaset
        ```

  * **spec**

    * Função: definir a especificação do objeto

    * Definir o **selector**

      * Sintaxe

        ```yml
        spec:
          selector:
            matchLabels:
              app: valor
        ```

      * Exemplo

        ```yml
        spec:
          selector:
            matchLabels:  
              app: nginx
        ```

    * Definir o **template**

      * Sintaxe

        ```yml
        spec:
          template:
            # configurações do Pod (metadados e specs)
        ```

      * Exemplo

        ```yml
        spec:
          template:
            metadata:
              labels:
                app: nginx
            spec:
              containers:
                - name: site
                  image: kubedevio/nginx-color:blue
                  ports:
                    - containerPort: 80
        ```

  * **Arquivo final**

    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: meureplicaset
    spec:
      replicas: 10
      selector:
        matchLabels:  
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
            - name: site
              image: kubedevio/nginx-color:green
              ports:
                - containerPort: 80
    ```

* **Verificação**

  * **Verificar a escalabilidade de um ReplicaSet**

    * Arquivo manifesto do ReplicaSet

      * Sintaxe

        ```yaml
        spec:
          replicas: valor
        ```

      * Exemplo
      
        ```yaml
        spec:
          replicas: 10
        ```

    * **OBS**: é possível definir o número de ReplicaSet através da linha de comando

      * Sintaxe: `$ kubectl scale replicaset <nome> --replicas <quantidade>`

      * Exemplo: `$ kubectl scale replicaset meureplicaset --replicas 20`

    * Recriar o ReplicaSet (`$ kubectl apply -f <arquivo manifesto>`)

  * **Verificar a resiliência do ReplicaSet**

    * Remover um das réplicas (Pod) criada pelo ReplicaSet (`$ kubectl delete pod <nome>`)

    * Verificar se o ReplicaSet criou um novo Pod para substituir o Pod removido (`$ kubectl get pods`)

* **Comandos**

  * **Criar o ReplicaSet**:

    * Sintaxe: `$ kubectl apply -f <arquivo manifesto>`

    * Exemplo: `$ kubectl apply -f ./meureplicaset.yml`

  * **Listagem dos ReplicaSets**: `$ kubectl get replicaset`

    * **DESIRED** = número de réplicas desejada

    * **CURRENT** = número de réplicas em execução

    * **READY** = número de réplicas prontas

    * **AGE** = tempo de vida (em segundos)

  * **Visualizar mais informações sobre um ReplicaSet**

    * Sintaxe: `$ kubectl describe replicaset <nome>`

    * Exemplo: `$ kubectl describe replicaset meureplicaset`

### Arquivo de manifesto de um **Deployment**

* **Campos obrigatórios na criação de um manifesto**

  * **apiVersion**
    
    * Função: definir o agrupamento de API, ou seja, a separação de alfa, beta, pertence a um recurso específico ou recurso final
    
    * Sintaxe: `apiVersion: app/valor`

    * Exemplo: `apiVersion: apps/v1`

    > Qual apiVersion utilizar? `$ kubectl api-resources`

  * **kind**

    * Função: definir o objeto a ser declarado

    * Sintaxe: `kind: valor`

    * Exemplo: `kind: Deployment`

  * **metadata**

    * Função: definir os metadados do objeto

    * Definir o nome do objeto

      * Sintaxe 

        ```yml
        metadata:
          name: valor
        ```

      * Exemplo

        ```yml
        metadata:
          name: meudeployment
        ```

  * **spec**

    * Função: definir a especificação do objeto

    * Definir o **selector**

      * Sintaxe

        ```yml
        spec:
          selector:
            matchLabels:
              app: valor
        ```

      * Exemplo

        ```yml
        spec:
          selector:
            matchLabels:  
              app: nginx
        ```

    * Definir o **template**

      * Sintaxe

        ```yml
        spec:
          template:
            # configurações do Pod (metadados e specs)
        ```

      * Exemplo

        ```yml
        spec:
          template:
            metadata:
              labels:
                app: nginx
            spec:
              containers:
                - name: site
                  image: kubedevio/nginx-color:blue
                  ports:
                    - containerPort: 80
        ```

  * **Arquivo final**

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: meudeployment
    spec:
      replicas: 10
      selector:
        matchLabels:  
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
            - name: site
              image: kubedevio/nginx-color:green
              ports:
                - containerPort: 80
    ```

* **Comandos**

  * **Criar um Deployment**

    * Sintaxe: `$ kubectl apply -f <arquivo manifesto>`

    * Exemplo: `$ kubectl apply -f ./meudeployment.yml`

  * **Listagem dos Deployments**: `$ kubectl get deployment`

    * **READY** = número de réplicas prontas

    * **UP-TO-DATE** = número de réplicas atualizadas

    * **AVAILABLE** = número de replicas disponíveis

    * **AGE** = tempo de vida (em segundos)

  * **Dar um rollback para versão anterior**

    * Sintaxe: `$ kubectl rollout undo deployment <nome>`

    * Exemplo: `$ kubectl rollout undo deployment meudeployment`

### Recursos: **label** e **selector**

* **labels**

  * **Definição**: São atributos "chave-valor" utilizados para marcação no Kubernetes

* **selectors**

  * **Definição**: selecionar objetos no _cluster_ Kubernetes que possuem um determinado **label** ("chave-valor"), ou seja, cumprem com uma regra de seleção

* Adicionar um **label** no arquivo manifesto de um **Pod**

  * **Sintaxe**

    ```yml
    metadata:
      name: valor
      labels:
        app: valor
    ```

  * **Exemplo**

    ```yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: meuseundopod
      labels:
        app: nginx
    spec:
      containers:
        - name: site
          image: kubedevio/nginx-color:blue
          ports:
            - containerPort: 80
    ```

* Visualizar o **Pod** com o um determinado label

  * **Sintaxe**: `$ kubectl get pods -l <label>`

  * **Exemplo**: `$ kubectl get pods -l app=nginx`

### Service Discovery no Kubernetes

* **Service do tipo _cluster_ IP**

  * **Definição**: expor um **Pod** (aplicação) apenas de forma interna, ou seja, só pode ser acessado dentro do _cluster_ Kubernetes

  * **Caso de uso**: loadbalancer em ambiente de nuvem

* **Service do tipo nodeport**

  * **Definição**: expor um **Pod** (aplicação) de forma externa, entretanto só pode ser acessado por máquinas que pertencem ao _cluster_ Kubernetes (node / master)

  * **Caso de uso**: ambientes Kubernetes on premise (não possui um sistema de nuvem que forneça um IP)

  * **OBS**: a escolha da porta é randômica, mas varia entre as portas 30000 e 32767 (**padrão**)

* **Arquivo manifesto**

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
  spec:
    selector:
      app: nginx
    ports:
      - port: 80
        nodePort: 30000
    type: NodePort
  ```

* **Binding de porta entre a maquina local e o container**

  * Arquivo manifesto

    * Sintaxe

      ```yaml
      spec:
        ports:
          nodePort: valor
      ```

    * Exemplo

      ```yaml
      spec:
        ports:
          - port: 80
            nodePort: 30000
      ```

  * **Criar cluster com binding de portas**

    * Sintaxe: `$ k3d cluster create --servers <número> --agents <número> -p "<porta maquina local>:<porta node>@loadbalancer"`

    * Exemplo: `k3d cluster create --servers 1 --agents 2 -p "8080:30000@loadbalancer"`

* **Comandos**

  * **Listagem dos Services**: `$ kubectl get services`

### Projeto 

* **Criação do Dockerfile na pasta /src**

  ```dockerfile
  FROM python:3.8-slim-buster

  WORKDIR /app

  COPY requirements.txt .

  RUN python -m pip install -r requirements.txt

  COPY . .

  EXPOSE 5000

  # subir o servidor HTTP
  CMD ["gunicorn", "--workers=3", "--bind", "0.0.0.0:5000", "app:app"]
  ```

* **Criação da imagem v1 a partir do Dockerfile**: `$ docker build -t imgabreuw/rotten-potatoes:v1 .`

* **Fazer autenticação no Docker Hub**: `$ docker login`

* **Fazer o push da imagem v1 para o Docker Hub**: `$ docker push imgabreuw/rotten-potatoes:v1`

* **Criação da imagem latest a da v1**: `$ docker tag imgabreuw/rotten-potatoes:v1 imgabreuw/rotten-potatoes:latest`

* **Criação do cluster Kubernetes local com k3d**: `$ k3d cluster create meucluster --servers 1 --agents 2 -p "8080:30000@loadbalancer"`

* **Criação do manifesto do Deployment para o MongoDB**

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: mongodb
  spec:
    selector:
      matchLabels:
        app: mongodb
    template: # especificações do Pod mongodb
      metadata:
        labels:
          app: mongodb
      spec:
        containers:
          - name: mongodb
            image: mongo:4.4.6
            ports:
              - containerPort: 27017
            env:
              - name: MONGO_INITDB_ROOT_USERNAME
                value: mongouser
              - name: MONGO_INITDB_ROOT_PASSOWORD
                value: mongopwd
    ```

  * `$ kubectl create -f ./k8s/mongodb/deployment.yml`

* **Criação do manifesto Service para o MongoDB**

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: mongo-service
  spec:
    selector:
      app: mongodb
    ports:
      - port: 27017
        targetPort: 27017
    type: ClusterIP
  ```

  * `$ kubectl apply -f ./k8s/mongodb/service.yml`

* **Criação do manifesto do Deployment e Service para a Web** (no mesmo arquivo)

  ```yml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: movies
  spec:
    selector:
      matchLabels:
        app: movies
    template:
      metadata:
        labels:
          app: movies
      spec:
        containers:
          - name: movies
            image: imgabreuw/rotten-potatoes:v1
            ports:
              - containerPort: 5000
            env:
              - name: MONGODB_DB
                value: admin
              - name: MONGODB_HOST
                value: mongo-service
              - name: MONGODB_PORT
                value: "27017"
              - name: MONGODB_USERNAME
                value: mongouser
              - name: MONGODB_PASSWORD
                value: mongopwd

  ---

  apiVersion: v1
  kind: Service
  metadata:
    name: movies-service
  spec:
    selector:
      app: movies
    ports:
      - port: 80
        targetPort: 5000
        nodePort: 30000
    type: NodePort
  ```

  * `$ kubectl apply -f ./k8s/web/deployment.yml`

* **Acessar**

  * Página inicial: http://localhost:8080/

  * Página para verificar a escalabilidade (nome do Pod): http://localhost:8080/host (`$ kubectl scale deployment movies --replicas <quantidade>`)