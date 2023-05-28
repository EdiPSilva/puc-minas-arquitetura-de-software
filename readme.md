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
* Escalabidade;
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
* Instancia EC2;
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
    * Recebe os parâmetros gerais de configuração do sistema, como unidade de medida sendo centimetros, metros, configuração de peso, entre outras.
* Invoicing
    * Recebe os dados de cálculo de frete e os fornecidos pela transportadora via arquivo/endpoint de conemb.
* Invoicing Operation
    * Realiza a conciliação de frete, fazendo um comparativo do cálculo realizado via TMS e pela transportadora.
* Order Data
    * Armazena e gerencia todos os dados do pedido
* Order Flow
    * Gerencia as movimentações dos pedidos durante a operação, como entrada na base, execução de cálculo, embarque, faturamento, etc.
* Packing List
    * Gerencia o romaneio e documento de embarque dos pedidos.
* Postal Code
    * Reponsável por validar e ser um repositório único de CEPs e endereços.
* Reports
    * Amazena os dados do TMS para gerar relatórios e execução de operações de BI.
* Shipping
    * Modulo de gerenciamento e configuração das transportadoras.
* Tracking
    * Gerencia os estágios do transporte do pedido, data de saída de entrega, etapas até o endereço e conclusão da entrega.

**SFTP:**<br>
Utilizando deste formato de arquitetura, a integração entre a transportadora poderá ser realizada de algumas formas:
* Enviando arquivos .txt no formato *Eletronic Data Interchange (EDI)* ao SFTP
    * Assim que subir o arquivo o AWS Transfer envia o arquivo ao Bucket S3.
    * Então as Lambda Function entra em ação e realiza a validação do layout do arquivo.
    * Caso tudo esteja correto os dados de ocorren ou conemb serão gravados na base.
    * Entretanto se houver algum problema será gerado um log na base não relacional.
* Utilizar do módulo da transportadora
    * Nesse formato a transportadora iria enviar o arquivo direto ao Bucket S3.
* Utilizar dos endpoint dia API Rest.
    * Deste modo não há necessidade de validar o layout do arquivo, porque a comunicação é no formato JSON.

**Comunicação externa:**<br>
No diagrama há algumas representações de comunicação esternas, que são sistemas e processos executados por aplicações fora do contexto do TMS, por exemplo:
* Transportadora
    * Faz uso do SFTP
    * Faz uso do Módulo da transportadora
* E-commerce
    * Realiza cotação de frete
* ERP
    * Fornece os dados do pedido
    * Recupera os dados de cálculo
    * Recupera os dados de faturamento
* WMS
    * Fornece os dados dos pedidos que serão transportados