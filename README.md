# Sistema de Gestão de Pedidos

## Desafio Tech Challenge - Fase 4 - FIAP

O sistema consiste em 4 serviços, separados em: produto, clientes, pedidos e logística. Este respositório os agrupa.

1. [Cliente](https://github.com/AydanAmorim/costumers-microservice)
2. [Gateway](https://github.com/fysabelah/gateway)
3. [Logística](https://github.com/erickmatheusribeiro/tracking-microservice)
4. [Pedido](https://github.com/fysabelah/ordering-microservice)
5. [Produto](https://github.com/DFaccio/products-microservice)
6. [Registro e descoberta de serviços](https://github.com/fysabelah/registration-discovery-services)

## Tecnologias
* Spring Boot para a estrutura do serviço
* Spring Data JPA para manipulação de dados dos pedidos
* Spring Cloud para comunicação baseada em eventos com outros microsserviços
* PostgreSQL para persistência
* RabbitMQ para mensageria

## Desenvolvedores

- [Aydan Amorim](https://github.com/AydanAmorim)
- [Danilo Faccio](https://github.com/DFaccio)
- [Erick Ribeiro](https://github.com/erickmatheusribeiro)
- [Isabela França](https://github.com/fysabelah)

## Esquema de funionamento entre os serviços

* Cadastro do cliente, onde mesmo pode está desabilitado ou habilitado.
* Cadastro dos produtos, onde um produto pode ter desconto ou não.
  * Para o pedido será necessário criar uma reserva 
* Cadastro das tarifas para gestão de logística.
* Após ambiente configurado, é possível partir para criação de pedido. O payload do pedido considera que a reserva dos sku já
foi criada. Conforme o pedido avança de status, a reserva será confirmada.
  * Para simulação do pagamento, será validado pelo último número do cartão, conforme [mencionado aqui](https://github.com/fysabelah/ordering-microservice/tree/main).
  * Um pedido pode ser cancelado a qualquer momento, contato que não esteja entregue.
    * Ele tentará cancelar a entrega e atualizar o estoque dependendo do status atual.
  * O mapa de status segue conforme imagem abaixo.
  
  ![Mapa de Status](mapa-status.jpeg)


## Acesso a documentação

Foi feito uso de um Gateway para comunicação dos serviços, para que assim toda a comunicação fosse realizada através do mesmo.
No entanto, as portas de cada aplicação estão expostas e também podem ser acessadas através deste. Porém, apenas o servidor 
do Gateway foi configurado diretamente no Swagger.

    As portas foram mantidas expostas devido a não centralização do Swagger e facilidades caso seja necessário algum
    desenvolvimento.

    Para acessar a documentação a aplicação precisa está executando.

* Clientes
  * [Swagger-UI](http://localhost:7076/doc/customer-management.html)
  * [Gateway](http://localhost:7071/order-management-system/customers-microservice/documentation)
* Pedidos
  * [Swagger-UI](http://localhost:7078/doc/order.html)
  * [Gateway](http://localhost:7071/order-management-system/ordering-microservice/documentation)
* Produtos
  * [Swagger-UI](http://localhost:7077/doc/product-management.html)
  * [Gateway](http://localhost:7071/order-management-system/product-microservice/documentation)
* Transporte
  * [Swagger-UI](http://localhost:7079/doc/tracking.html)
  * [Gateway](http://localhost:7071/order-management-system/tracking-microservice/documentation)


## Como executar

1. Clone o repositório
2. No diretório root do projeto, crie um arquivo .env

       PROFILE=prod

       # Eureka
       EUREKA_SERVER=http://server-discovery:7070/eureka
       
       # RabbitMQ
       RABBITMQ_USER=message_admin
       RABBITMQ_PASS=senha_para_rabbitmq
       
       # PostgreSQL
       DATABASE_USERNAME=postgres
       DATABASE_PASSWORD=senha_para_postgre
       DATABASE_HOST=postgres
       
       #PgAdmin
       PGADMIN_DEFAULT_EMAIL=order@gmail.com
       PGADMIN_DEFAULT_PASSWORD=senha_para_pgadmin
       
       # Gateway
       CUSTOMER_ADDRESS=lb://customers-microservice
       ORDER_ADDRESS=lb://ordering-microservice
       LOGISTICS_ADDRESS=lb://tracking-microservice
       INVENTORY_ADDRESS=lb://product-microservice

3. Execute o comando abaixo

       docker compose up

Caso prefira, também é possível executá-los em uma IDE de sua preferência.

É possível ver todos serviços registrados, considerando que nada foi alterado, na porta [7070](http://localhost:7070).

### Novos desenvolvimento

Para novos desenvolvimentos basta executar o projeto conforme mencionado acima. Considerando que irá executar o serviço
localmente, altere a variável dentro da `Gateway` do arquivo conforme o serviço que irá alterar. 

    O valor deve ser alterado antes da executação do passo 3. 

* Cliente: http://host.docker.internal:7076
* Pedido: http://host.docker.internal:7078
* Produto: http://host.docker.internal:7077
* Transporte: http://host.docker.internal:7077

Exemplo: considerando que irá mexer no serviço de clientes, então: CUSTOMER_ADDRESS = http://host.docker.internal:7076,
resultando:

    PROFILE=prod

    # Eureka
    EUREKA_SERVER=http://server-discovery:7070/eureka
    
    # RabbitMQ
    RABBITMQ_USER=message_admin
    RABBITMQ_PASS=senha_para_rabbitmq
    
    # PostgreSQL
    DATABASE_USERNAME=postgres
    DATABASE_PASSWORD=senha_para_postgre
    DATABASE_HOST=postgres
    
    #PgAdmin
    PGADMIN_DEFAULT_EMAIL=order@gmail.com
    PGADMIN_DEFAULT_PASSWORD=senha_para_pgadmin
    
    # Gateway
    CUSTOMER_ADDRESS=http://host.docker.internal:7076
    ORDER_ADDRESS=lb://ordering-microservice
    LOGISTICS_ADDRESS=lb://tracking-microservice
    INVENTORY_ADDRESS=lb://product-microservice

### Curiosidades sobre o arquivo de configuração

* PROFILE
  * Necessário antentar-se caso seja alterado. Alguns dos serviços mudam suas configurações de acordo com este.
  Para executar tudo em container, é recomendado `prod`. Também há o peril `dev`.
* Eureka
  * Este é o endereço do serviço que registra os serviços listados no gateway. Se ao menos um serviço irá executar em docker
  deixe como está. 
  * Serviços que não executam em docker usam localhost:7070. Mais informações de configuração no próprio projeto
* RabbitMQ
  * Usuário e senha para acesso ao RabbitMQ
* PostgreSQL
  * Usuário e senha para o Postgre
* PgAdmin
  * Usuário e senha para o PgAdmin
* Gateway
  * Dado a necessidade de desenvolvimento/debug, algum momento alguns serviços podem está sendo executados localmente e 
  por isto as variáveis foram criadas.
  * Quando tudo local ou tudo em docker, não há necessidade de alterar os valores. Porém, caso queira rodar local, basta
  alterar o endereço do serviço conforme mencionado em novos desenvolvimentos.
    * Dica: Execute `docker compose up` e pause o container que irá executar local
