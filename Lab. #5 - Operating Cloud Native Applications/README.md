# Lab. #5 - Operating Cloud Native Applications

Olá, neste laboratório você aprenderá como registrar, monitorar e analisar os logs da infraestrutura Compute de OCI que você provisionou nos laboratórios anteriotes utilizando a **Oracle Cloud Observability and Management Platform**!

- 🌀 [Página oficial do OCI Observability and Management Platform](https://www.oracle.com/br/manageability/)
- 🧾 [Documentação do OCI Logging](https://docs.oracle.com/en-us/iaas/Content/Logging/Concepts/loggingoverview.htm)
- 🧾 [Documentação do OCI Logging Analytics](https://docs.oracle.com/en-us/iaas/logging-analytics/index.html)

**A seguir você aprenderá o passo-a-passo desta configuração:**

- [Pre Reqs: Logar na sua conta OCI](#PreReqs)
- [Passo 1: Ativar o serviço de Logging e habilitar a coleta dos Logs](#Passo1)
- [Passo 2: Ativar o serviço de Logging Analytics e criar um grupo para os Logs](#Passo2)
- [Passo 3: Criar o Service Connector para replicar os logs do Logging para o Logging Analytics](#Passo3)
- [Passo 4: Configurar queries customizadas e criar um dashboard](#Passo4)

- - -

## <a name="PreReqs"></a> Pre Reqs: Criar Canal no Slack e logar na sua conta OCI

 1. Faça o [login](https://www.oracle.com/cloud/sign-in.html) em sua conta na OCI;
 2. Execute o [Lab. #1](../Lab.%20%231%20-%20Resource%20Provisioning), caso não o tenha executado anteriormente;

---

## <a name="Passo1"></a> Passo 1: Ativar o serviço de Logging e habilitar a coleta dos Logs

1. No 🍔 menu de hambúrguer, acesse: **Observability and Management Platform** → **Logging**:
![](https://github.com/ladan19/images-lp/blob/main/photo-2.png?raw=true)
2. No menu à esquerda **Logging** clique em **Logs** e em seguida no botão à direita **Enable service log**:
![](https://github.com/ladan19/images-lp/blob/main/photo-3.png?raw=true)
3. Escolha em **Service** o item *Virtual Cloud Network* e em **Resource** selecione a subnet pública criada anteriormete. Em **Log Category** selecione a opção de *Flow Logs* e em **Log Name** digite o nome *Flowlogs-VCN*. Depois em Log Location clique em **Show Advanced Options** e clique em **Create New Group** para criar um novo grupo:
![](https://github.com/ladan19/images-lp/blob/main/photo-4.png?raw=true)
4. Na tela de criação de grupo de log em **Name** digite o nome do grupo *LogGroupFlow* e clique no botão **Create**:
![](https://github.com/ladan19/images-lp/blob/main/photo-5.png?raw=true)
5. Deixe selecionado o *LogGroupFlow* como **Log Group** e clique no botão **Enable Log** para habilitar a configuração:
![](https://github.com/ladan19/images-lp/blob/main/photo-6.png?raw=true)
6. Após a ativação (2-3 min), inicia-se a coleta de logs (5-6 min). Para visualizar no menu à esquerda **Logging** clique em **Logs** e depois clique no Log Name que acabamos de criar **Flowlogs-VCN**:
![](https://github.com/ladan19/images-lp/blob/main/photo-7.png?raw=true)
7. Você vizualizará o dashboard de coleta de logs da VCN escolhida. Clique em **Explore with Log Search** à direita para:
![](https://github.com/ladan19/images-lp/blob/main/photo-8.png?raw=true)
8. Pronto! A pardir de agora você pode modificar as buscar para filtrar o log desejado.

> Dica: Mude a vialização para **Visualize** e divirta-se!


---

## <a name="Passo2"></a> Passo 2: Ativar o serviço de Logging Analytics e criar um grupo para os Logs

1. No 🍔 menu de hambúrguer, acesse: **Observability and Management Platform** → **Logging Analytics** :
![](https://github.com/ladan19/images-lp/blob/main/photo-10.png?raw=true)
2. Ative o clicando no botão **Start Using Logging Analytics**:
![](https://github.com/ladan19/images-lp/blob/main/photo-11.png?raw=true)
3. Após a inicialização, clique no botão **Take me to Log Explorer**:
![](https://github.com/ladan19/images-lp/blob/main/photo-12.png?raw=true)

> Dica: Repare que o serviço já cria algumas poíticas e um grupo de log *Default*.

5. Na console de Log Explorer no menu superior à esquerda clique e selecione **Administration**:
![](https://github.com/CeInnovationTeam/OCI-FastTrack-Developer-LINUXtips/blob/main/Lab.%20%235%20-%20Operating%20Cloud%20Native%20Applications/images/Image02.png?raw=true)
6. Agora clique em **Log Groups** no menu **Resources** e em seguida no botão **Create Log Group**, para criarmos um novo grupo de log:
![](https://github.com/ladan19/images-lp/blob/main/photo-13.png?raw=true)
![](https://github.com/ladan19/images-lp/blob/main/photo-14.png?raw=true)
7. Na console de criação de grupo de Log em **Name** digite o nome do grupo *LogGroupVCN* e depois clique no botão **Create**:
![](https://github.com/ladan19/images-lp/blob/main/photo15.png?raw=true)

---

## <a name="Passo3"></a> Passo 3: Criar o Service Connector para replicar os logs do Logging para o Logging Analytics

1. No 🍔 menu de hambúrguer, acesse: **Observability and Management Platform** → **Service Connectors**:
![](https://github.com/ladan19/images-lp/blob/main/photo-16.png?raw=true)
3. Na console de *Service Connectors* clique no botão **Create Service Connector**:
![](https://github.com/CeInnovationTeam/OCI-FastTrack-Developer-LINUXtips/blob/main/Lab.%20%235%20-%20Operating%20Cloud%20Native%20Applications/images/Image03.png?raw=true)
1. Em **Connector Name** digite *LogVCNConnector*, em **Configure Source** selecione *Logging* e em **Target** selecione *Logging Analytcs*. Na parte de *Configure Source* selecione em **Log Group** o *LogGroupFlow* e em **Logs** selecione o *FlowLogs-VCN* criados anteriosmente:
![](https://github.com/ladan19/images-lp/blob/main/photo17.png?raw=true)
1. Em **Configuration Target** selecione o **Log Group** *LogGroupVCN* e (Muito Importante :warning:) clique no botão **Create** à direita _para criar as políticas para o conector tenha permissão de escrita_. Após isso clique no botão **Create** no canto inferior à esquerda para criar o conector:
![](https://github.com/ladan19/images-lp/blob/main/photo-18.png?raw=true)

---

## <a name="Passo4"></a> Passo 4: Configurar queries customizadas e criar um dashboard

1. No 🍔 menu de hambúrguer, acesse: **Observability and Management Platform** → **Log Explorer**:
![](https://github.com/ladan19/images-lp/blob/main/photo-19png.png?raw=true)
2. Na console **Log Explorer** substitua a query existente pela query abaixo para buscar os IPs de origem que estão acessando a VCN que configuramos e clique no botão **Run**:

```sh
'Log Source' = 'OCI VCN Flow Unified Schema Logs' | stats count as logrecords by 'Source IP'
```

![](https://github.com/ladan19/images-lp/blob/main/photo-21.png?raw=true)

> Dica: Caso não apareça a mensagem *No data has been ingested* no **Log Explorer**, clique no menu à esquerda que está escrito **Logging Analytics** → **Home** e clique na imagem **VCN Flow Logs**.

3. Salvaremos o resultado da query para utilizarmos na criação do nosso dashboard a seguir. Clique em **Actions** no menu à direita e em **Save**, digite *Ips de Entrada* em **Search Name** e clique no botão **Save**:
![](https://github.com/CeInnovationTeam/OCI-FastTrack-Developer-LINUXtips/blob/main/Lab.%20%235%20-%20Operating%20Cloud%20Native%20Applications/images/Image06.png?raw=true)
4. Configure outra query customizada para saber o volume do tráfego de saída da VCN. substitua a query existente pela query abaixo, troque a visualização para gráfico de **Line** e clique no botão **Run**:

```sh
'Log Source' = 'OCI VCN Flow Unified Schema Logs' | timestats avg('Content Size Out') as 'Outbound Traffic'
```

![](https://github.com/ladan19/images-lp/blob/main/photo-23.png?raw=true)

5. Clique em **Actions** no menu à direita e em **Save as...**, digite *Tráfego de Saída* em **Search Name** e clique no botão **Save**:
![](https://github.com/CeInnovationTeam/OCI-FastTrack-Developer-LINUXtips/blob/main/Lab.%20%235%20-%20Operating%20Cloud%20Native%20Applications/images/Image05.png?raw=true)

> Dica: Utilizar **Save as..** ao invés de **Save** para conseguir salvar o resultado com um novo nome.

6. No menu ao lado esquerdo superior selecione **Dashboard**:
![](https://github.com/ladan19/images-lp/blob/main/photo-26.png?raw=true)
7. Na console de **Dashboard** clique no botão **Create Dashboard**:
![](https://github.com/ladan19/images-lp/blob/main/photo-27.png?raw=true)
8. Na console de criação do dashboard, selecione o compartement em **Widget Compartment** e arraste e solte o widget **Ips de Entrada**:
![](https://github.com/ladan19/images-lp/blob/main/photo-28.png?raw=true)
10. Após arrastar o widget, será solicitado a criação do filtro. Adicionaremos um novo filtro, deixe a seleção **Log Group Compartment** e clique no botão **Save Changes**:
![](https://github.com/CeInnovationTeam/OCI-FastTrack-Developer-LINUXtips/blob/main/Lab.%20%235%20-%20Operating%20Cloud%20Native%20Applications/images/Image07.png?raw=true)
11. Para a configuração da **Entity**, deixe a seleção *Entity* e clique no botão **Save Changes**:
![](https://github.com/CeInnovationTeam/OCI-FastTrack-Developer-LINUXtips/blob/main/Lab.%20%235%20-%20Operating%20Cloud%20Native%20Applications/images/Image08.png?raw=true)
12. O widget será adicionado ao dashboard dessa maneira:
![](https://github.com/ladan19/images-lp/blob/main/photo-30.png?raw=true)
13. Realize o mesmo processo realizado anterioemnte clicando na aba **Add widget** para o widget **Tráfego de Saída**:
![](https://github.com/ladan19/images-lp/blob/main/photo-31.png?raw=true)
14. Modifique no nome dashboard clicando no ícone **Pencil**, digite *VCN Dashboard* e tecle enter para salvar o nome. Após isso selecione a aba **About**, selecione um compartment e clique no botão **Save Changes**:
![](https://github.com/ladan19/images-lp/blob/main/photo-33.png?raw=true)

---

### 👏🏻 Parabéns!!! Você foi capaz de configurar com sucesso um pipeline completo de **Logging** e **Logging Analytics** em OCI! 🚀
