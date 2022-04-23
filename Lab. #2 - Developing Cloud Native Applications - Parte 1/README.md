# Lab 2 - Developing Cloud Native Applications - Parte 1

Laboratório para mostrar na prática o funcionamento das ferramentas de developer no OCI.

## Objetivo

Criar uma aplicação no Kubernetes com as imagens de container armazenadas no Oracle Container Registry (OCIR). O backend da aplicação será exposto através do API Gateway, onde receberá os headers de CORS necessários para se comunicar com o frontend.

Além disso a aplicação já contará com as bibliotecas e configurações necessárias para ser monitorada pelo APM que será demonstrado no laboratório 5.

- [Lab 2 - Developing Cloud Native Applications - Parte 1](#lab-2---developing-cloud-native-applications---parte-1)
  - [Objectivo](#objectivo)
  - [Coleta de Informações](#coleta-de-informações)
    - [Tenancy Namespace](#tenancy-namespace)
    - [User OCID & Auth Token](#user-ocid--auth-token)
    - [Dados do APM](#dados-do-apm)
    - [Código da Região](#código-da-região)
  - [Docker Login](#docker-login)
  - [Configurar o Kubectl](#configurar-o-kubectl)
  - [Copiar o Código](#copiar-o-código)
  - [Configurar e fazer o Deploy do Backend](#configurar-e-fazer-o-deploy-do-backend)
    - [Docker Build](#docker-build)
    - [Docker Push](#docker-push)
    - [Criando Secret no Kubernetes](#criando-secret-no-kubernetes)
    - [Configurar o Manifesto de Kubernetes](#configurar-o-manifesto-de-kubernetes)
    - [Deploy no Kubernetes](#deploy-no-kubernetes)
  - [Configuração API Gateway](#configuração-api-gateway)
    - [Deployment](#deployment)
  - [Configurar e fazer Deploy do Frontend](#configurar-e-fazer-deploy-do-frontend)
    - [Configurando o Frontend](#configurando-o-frontend)
    - [Docker Build Front](#docker-build-front)
    - [Docker Push Front](#docker-push-front)
    - [Configurar o Manifesto do Kubernetes](#configurar-o-manifesto-do-kubernetes)
    - [Deploy do Front no Kubernetes](#deploy-do-front-no-kubernetes)
  - [Testando a Aplicação](#testando-a-aplicação)

## Coleta de Informações

Vamos coletar algumas informações na tenancy do OCI que serão utilizadas ao logo do laboratório, recomendamos que as anote em um bloco de nota para ter sempre em mãos de modo fácil. Serão coletadas as seguintes informações:

```bash
Tenancy Namespace:
User Name:
Auth Token:
APM Endpoint:
Public Key:
Código da Região:
```

### Tenancy Namespace

Clique no menu do lado direto no icone do usuário, clique no nome da sua tenency.

![namespace](images/namespace1.png)

Agora copie o namespace para o bloco de notas.

![namespace](images/namespace2.png)

### User OCID & Auth Token

Clique no menu do lado direto no icone do usuário, clique no nome do seu usuário.

![user](images/user1.png)

Copie o OCID do usuário e salve no bloco de notas.

Depois, vá em Auth Tokens e gere um novo token, salve o token no bloco de notas.

![user](images/user2.png)

### Dados do APM

Navegue no menu principal em Observability & Management > Application Monitoring> Administration

Clique no domínio criado pelo Resource Manager no laboratório anterior, e copie os dados do Endpoint e da Public Key.

![apm](images/apm.png)

### Código da Região

Você pode pesquiar o código da sua região [aqui](https://docs.oracle.com/en-us/iaas/Content/Registry/Concepts/registryprerequisites.htm#regional-availability)

## Docker Login

Vamos precisar do Docker para fazer o build dos containers da aplicação e fazer o push para o OCIR. Antes do push, precisamos nos logar no OCIR através do dorcker-CLI.

Abra o **Cloud Shell** e execute o comando abaixo substituindo o username, tenanacy ocid e código da região. E na senha utilize o Auth Token gerado anteriormente.

```bash
docker login <Codigo Region>.ocir.io -u <tenancy-namespace>/<username>
```

Resultado:

```bash
password: <Auth Token>
WARNING! Your password will be stored unencrypted in /home/trial01oci/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
```

## Configurar o Kubectl

Agora vamos configurar o acesso ao Kubernetes via Kubectl no Cloud Shell, no menu principal vá em **Developer Services > Containers & Artifacts > Kubernetes Clusters (OKE)**.

Entre no cluster criado via Resource manager e clique no botão **Access Cluster**

![oke](images/oke1.png)

Copie o comando que aparece no popup e execute no cloud shell.

Exemplo:

```bash
$ oci ce cluster create-kubeconfig --cluster-id ocid1.cluster.oc1.sa-saopaulo-1.aaaaaaaan2pf --file $HOME/.kube/config --region sa-saopaulo-1 --token-version 2.0.0  --kube-endpoint PUBLIC_ENDPOINT

New config written to the Kubeconfig file /home/trial01oci/.kube/config

```

O acesso pode ser testado com o seguinte comando:

```bash
kubectl get nodes
```

Deve ter uma resposta parecida com essa:

```bash
NAME           STATUS   ROLES   AGE     VERSION
10.20.10.125   Ready    node    3h23m   v1.21.5
10.20.10.138   Ready    node    3h23m   v1.21.5
10.20.10.208   Ready    node    3h23m   v1.21.5
```

## Copiar o Código

Abra o Cloud Shell e execute o git clone do código da aplicação:

```bash
git clone https://github.com/ChristoPedro/labcodeappdev.git 
```

## Configurar e fazer o Deploy do Backend

Navegue até a pasta do backend:

```bash
cd labcodeappdev/Backend/code
```

Vamos realizar o build da imagem do backend e depois fazer o push para o OCIR.

### Docker Build

Execute o comando:

```bash
docker build -t <Codigo Region>.ocir.io/<tenancy-namespace>/ftdeveloper/back .
```

### Docker Push

Depois da Build vamos fazer o push para o OCIR

```bash
docker push <Codigo Region>.ocir.io/<tenancy-namespace>/ftdeveloper/back
```

### Criando Secret no Kubernetes

Vamos criar um secret que irá conter as informações do login do OCIR. Permitindo assim que seja feito o pulling das images.

Basta executar esse código, substituindo os valores

```bash
kubectl create secret docker-registry ocisecret --docker-server=<region-key>.ocir.io --docker-username='<tenancy-namespace>/<oci-username>' --docker-password='<oci-auth-token>' --docker-email='<email-address>'
````

Resposta:

```bash
secret/ocisecret created
```

### Configurar o Manifesto de Kubernetes

Vamos agora voltar uma pasta:

```bash
cd ..
```

Editar o código para adicionar os parametros do APM e da imagem:

```bash
vi Deploybackend.yaml
```

Pressione **i** para editar.

Substitua os valores de **Image-Name**, **Endpoint do APM** e **Key do APM** nas seguites linhas:

```note
Image-Name = <Codigo Region>.ocir.io/<tenancy-namespace>/ftdeveloper/back
```

```yaml
      - name: backend
        image: [Image-Name]:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 5000
        env:
        - name: APM_URL
          value: "[Substitua pelo Endpoint do APM]"
        - name: APM_KEY
          value: "[Substitua pela Public Key do APM]"
```

Após substituir os valores utilize os seguintes comando **ESC : WQ** e pressione Enter.

### Deploy no Kubernetes

Com o arquivo editado podemos executar o seguinte comando para realizar o deploy:

```bash
kubectl apply -f Deploybackend.yaml
```

Deve ter uma saida como a seguinte:

```bash
deployment.apps/cepapp-backend created
service/cepapp-backend created
```

Podemos usar o seguinte código para saber se os pods já estão no ar:

```bash
kubectl get pods
```

## Configuração API Gateway

Primeiro precisamos descobrir o IP do **Load Balancer** do serviço do backend.

```bash
kubectl get svc cepapp-backend
```

A resposta será parecida com essa:

```bash
NAME             TYPE           CLUSTER-IP      EXTERNAL-IP    PORT(S)          AGE
cepapp-backend   LoadBalancer   10.96.123.143   10.20.20.237   5000:31952/TCP   13m
```

Vamos utilizar o EXTERNAL-IP para realizar expor-lo através do **API Gateway**.

Agora vamos navegar no menu principal **Developer Services > API Management > Gateways**. E selecionar o gateway já criado pelo Resource Manager. No menu do lado esquerdo vamos em Deploymets.

![apigw](images/api1.png)

E agora vamos criar um novo deployment, que conterá a rota do backend que será consumida pelo frontend.

### Deployment

Preencha as informações básicas com os sequintes dados:

- **Name**: backend
- **PATH PREFIX**: /cep
- **Compartment**: Selecione seu compartimento.

![apigw](images/api2.png)

Agora vamos preencher as informações de **CORS**, sem elas vamos ter erros nas chamadas entre o Frontend e o Backend.

Na região do CORS clique no botão add e preencha os seguintes campos:

- **ALLOWED ORIGINS**: *
- **Methods**: GET

E aplique as modificações.

![apigw](images/api3.png)

Com o CORS configurado, podemos clicar em **Next** e configurar a rota. Vamos preencher os campos da Rota1 da seguinte forma:

- **PATH**: /getcep
- **METHODS**: GET
- **TYPE**: HTTP
- **URL**: ```http://[External-IP-do-LoadBalancer]:5000```

![apigw](images/api4.png)

Depois de preenchido clique em **Next** e depois **Create**.

Quando a criação do Deployment estiver concluida, copie o URL do endpoint e teste a rota.

![apigw](images/api5.png)

Basta jogar o endpoint no navegador, no seguinte formato:

```bash
<seu_endpoint>/getcep?cep=<cep-da-sua-casa>
```

![apigw](images/api6.png)

## Configurar e fazer Deploy do Frontend

Para o Frontend precisamos substituir o URL do backend e as informações do APM antes de fazer a build do Docker.

### Configurando o Frontend

Vamos navegar até a pasta do javascript:

```bash
cd $HOME/labcodeappdev/Frontend/code/js
```

E editar o arquivo **api.js**

```bash
vi api.js
```

Vamos substituir a variável url.

```js
const url = '[Substituia com a URL do API Gateway]'
```

Para isso pressione **i** para editar o arquivo substitua as informações dentro das aspas.

```js
const url = 'https://ghstpnks2qut3htj2w7zmdtghi.apigateway.sa-saopaulo-1.oci.customer-oci.com/cep/getcep'
```

Para salvar use as teclas **ESC : WQ** .

Agora precisamos configurar o APM no HTML, vamos voltar uma pasta:

```bash
cd ..
```

E editar o arquivo **index.html**:

```bash
vi index.html
```

E substituir os valores nas seguintes linhas:

```html
<script>
  window.apmrum = (window.apmrum || {}); 
  window.apmrum.serviceName='CEP';
  window.apmrum.webApplication='cepapp';
  window.apmrum.ociDataUploadEndpoint='[Substitua com o Endpoint do APM]';
  window.apmrum.OracleAPMPublicDataKey='[Substitua com a Public Key do APM]';
</script>
<script async crossorigin="anonymous" src="[Substitua com o Endpoint do APM]/static/jslib/apmrum.min.js"></script>
```

Salve o arquivo.

### Docker Build Front

Após, configurar o frontend, vamos realizar a build do docker com o seguinte comando.

```bash
docker build -t <Codigo Region>.ocir.io/<tenancy-namespace>/ftdeveloper/front .
```

### Docker Push Front

Ao final da build podemos fazer o push para o OCIR

```bash
docker push <Codigo Region>.ocir.io/<tenancy-namespace>/ftdeveloper/front
```

### Configurar o Manifesto do Kubernetes

Agora precisamos voltar mais uma pasta:

```bash
cd ..
```

E editar o arquivo Deployfrontend.yaml:

```bash
vi Deployfrontend.yaml
```

 Pressione **i** para editar o arquivo, e substitua a **Image-Name**:

 ```note
Image-Name = <Codigo Region>.ocir.io/<tenancy-namespace>/ftdeveloper/front
```

 ```yaml
     spec:
      containers:
      - name: front
        image: [Image-Name]:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 80
      imagePullSecrets:
```

Após a alteração salve o arquivo com **ESC : WQ**.

### Deploy do Front no Kubernetes

Agora vamos executar o deploy do frontend no Kubernetes com o seguinte comando:

```bash
kubectl apply -f Deployfrontend.yaml
```

Resultado:

```bash
deployment.apps/cepapp-front created
service/cepapp-front created
```

## Testando a Aplicação

Agora com o deploy do frontend e do backend podemos testar a aplicação.

Vamos obter o IP do Load Balancer do Frontend para acessar a aplicação:

```bash
kubectl get svc cepapp-front
```

Obtendo um resultado parecido com esse:

```bash
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
cepapp-front   LoadBalancer   10.96.188.10   152.70.213.248   80:31117/TCP   89s
```

Basta copiar o IP externo no navegador e testar se aplicação retorna as informações.

![teste](images/teste.png)
