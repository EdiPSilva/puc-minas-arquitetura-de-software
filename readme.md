#TMS - Transport Management System
---
> [**Pós Graduação Arquitetura de Software**](https://www.pucminas.br/PUCVIRTUAL/Pos-Graduacao/Paginas/Arquitetura-de-Software-Distribuido.aspx?PageID=409&moda=1&polo=1&curso=2096&situ=1)<br/>
> [*PUC - Minas*](https://www.pucminas.br/pucvirtual/Paginas/default.aspx)
---
Este repositório detém do objetivo de centralizar os documentos gerados durante a confecção do trabalho e também proporcionar um detalhamento sobre um material extra que foi desenvolvido.<br/>
Durante o desenvolvimento do projeto foi desenvolvido duas APIs, e consultar um melhor detalhamento, basta clicar nos links da lista abaixo:
* [Data Calculate Shipping](https://github.com/EdiPSilva/data-calculate-freight)
    * *Responsável por gerir o cadastrameto dos dados*
* [Calculate Shipping](https://github.com/EdiPSilva/calculate-freight)
    * *Responsável por realizar o cálculo de frete*

---
**Diagrama:** Contexto
*Representa a versão atual do TMS.*
![Verão atual do TMS](/diagramas/contexto.png "Verão atual do TMS")
**Exemplo de alguns problemas:**
* Performance;
* Escalabilidade;
* Custo.
---
**Diagrama:** Container
*Demonstra as aplicações que serão criadas para inciar a solução do problema do cliente.*
![Verão atual do TMS](/diagramas/container.png "Verão atual do TMS")
**Aplicações desenvolvidas:**
* Dados calcula frete;
* Calcula frete.
---
**Diagrama:** Container
*Detalhamento do comportamento das aplicações.*
![Verão atual do TMS](/diagramas/component.png "Verão atual do TMS")

---
**Diagrama:** Nova arquitetura
*Levantamento dos novos microsserviços.*
![Verão atual do TMS](/diagramas/full_new_architecture.png "Verão atual do TMS")
Este diagrama detém do levantamento dos próximos passos que poderão ser executado com base na prioridade retornada pelo cliente.

**Componente AWS:** <br>
Será utilizado da cloud da AWS e seus componentes como:
* API Gateway;
* Load Balance;
* Cluster Kubernetes EKS;
* Banco de dados relacional RDS;
* Banco de dados não relacional DynamoDB;
* Bucket S3;
* Cloud Front;
* AWS Transfer;
* Instância EC2;
* Lambda Fuctions.

**Microsserviços:**
* Authentication Authorization
    * Responsável por autenticar e autorizar as requisições.
* Calculate Freight
    * Realiza o cálculo de frete.
* Data Calculate Freight
    * Armazena os parâmetros para cálculo de frete, como rotas, faixas de cep, etc.
* Freight Simulator
    * Realiza a simulação de cálculo de frete.
* General Parameters
    * Recebe os parâmetros gerais de configuração do sistema, como unidade de medida sendo centímetros, metros, configuração de peso, entre outras.
* Invoicing
    * Recebe os dados de cálculo de frete e os fornecidos pela transportadora via arquivo/endpoint de conemb.
* Invoicing Operation
    * Realiza a conciliação de frete, fazendo um comparativo do cálculo realizado via TMS e pela transportadora.
* Order Data
    * Armazena e gerencia todos os dados do pedido.
* Order Flow
    * Gerencia as movimentações dos pedidos durante a operação, como entrada na base, execução de cálculo, embarque, faturamento, etc.
* Packing List
    * Gerencia o romaneio e documento de embarque dos pedidos.
* Postal Code
    * Responsável por validar e ser um repositório único de CEPs e endereços.
* Reports
    * Armazena os dados do TMS para gerar relatórios e execução de operações de BI.
* Shipping
    * Módulo de gerenciamento e configuração das transportadoras.
* Tracking
    * Gerencia os estágios do transporte do pedido, data de saída de entrega, etapas até o endereço e conclusão da entrega.

**SFTP:**<br>
Utilizando deste formato de arquitetura, a integração entre a transportadora poderá ser realizada de algumas formas:
* Enviando arquivos .txt no formato *Eletronic Data Interchange (EDI)* ao SFTP
    * Assim que subir o arquivo o AWS Transfer envia o arquivo ao Bucket S3.
    * Então as Lambdas Function entrão em ação e realizão a validação do layout dos arquivos.
    * Caso tudo esteja correto os dados de ocorren ou conemb serão gravados na base.
    * Entretanto se houver algum problema será gerado um log na base não relacional.
* Utilizar do módulo da transportadora
    * Nesse formato a transportadora iria enviar o arquivo direto ao Bucket S3.
* Utilizar dos endpoint dia API Rest.
    * Deste modo não há necessidade de validar o layout do arquivo, porque a comunicação é no formato JSON.

**Comunicação externa:**<br>
No diagrama há algumas representações de comunicação externas, que são sistemas e processos executados por aplicações fora do contexto do TMS, por exemplo:
* Transportadora
    * Faz uso do SFTP.
    * Faz uso do Módulo da transportadora.
* E-commerce
    * Realiza cotação de frete.
* ERP
    * Fornece os dados do pedido.
    * Recupera os dados de cálculo.
    * Recupera os dados de faturamento.
* WMS
    * Fornece os dados dos pedidos que serão transportados.
---
**Diagrama:** Fluxo do TMS com a nova arquitetura
*Representação de parte do fluxo do TMS.*
![Fluxo do TMS](/diagramas/flow_tms.png "Fluxo do TMS")
Este diagrama detém de uma representação de parte do fluxo do TMS.
Na parte superior é notado a entrada de dados provocada pelo E-commerce durante as cotações de frete, pelo ERP com a entrada do pedido e nota fiscal e pelo WMS com os pedidos que serão embarcados.<br>
Durante as cotações de frete um dos procedimentos que será realizado é validar o CEP em um microsserviço, esta aplicação será uma base centralizada de endereços, provendo CEP e endereços válidos, junto a faixa de CEP por cidade, facilitando os cadastros no futuro.<br>
Junto a cotação também é possível identificar o simulador de frete, que permite que o usuário da aplicação realize testes de uma configuração de cálculo para um formato específico de produto.<br>
Um passo adiante, notamos a aplicação que será responsável por gerir todos os dados dos pedidos, como número, nota fiscal, dados do cliente como endereço, entre outros.<br>
A partir do fluxo do WMS é localizado a aplicação que irá gerir os embarques dos pedidos no romaneio.<br>
E posterior a essa etapa identificamos o microsserviço que irá gerir o tracking dos pedidos, que são as etapas do transporte, como por exemplo:
1. O pedido X entrou em transporte;
2. O pedido X está na rota Y;
3. O pedido X entá o centro de distribuição da cidade Y;
4. O pedido X está em rota de entrega;
5. Pedido X entregue na data Y.

Quando for constatado que a entrega foi realizada, poder ser efetuado o faturamento. Que recebe os dados de cálculo do TMS e realiza um comparativo sobre os valores de cálculo executado pela transportadora, efetuando então a conciliação de frete.<br>
Por trás de todos esses passos teremos a aplicação que lida com o fluxo do pedido em operação, que mostra ao usuário do sistema em que posição o pedido se encontra dentro do TMS, mostrando quando ele foi cadastrado, quando foi efetuado o cálculo, em que momento o pedido entrou em transporte e se já foi faturado.<br>
E não menos importante, o microsserviço de relatórios irá receber todas as movimentações das aplicações via kafka alimentando uma base centralizada para execução de relatórios e procedimentos de *Business intelligence (BI)*.