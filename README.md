# O que é REST (Representational State Transfer)?

É um estilo de arquitetura de software utilizado para criação de serviços web.
Define um conjunto de princípios e restrições que permitem que o sistemas se comuniquem de forma padronizada e escalável.

Em uma arquitetura REST, os recursos são identificados por URLs e utiliza métodos padrão do protocolo HTTP, como GET, POST, PUT e DELETE, para manipula-los.

Princípios-chave do REST incluem:

1. Cliente-Servidor: As duas partes são separadas.

2. Comunicação Stateless: Cada solicitação de um cliente para o servidor contém todas as informações necessárias para entender e processar a requisição. 
O servidor não mantém nenhum estado da sessão do cliente entre as solicitações.

3. Cacheable: O cliente deve ser informado sobre as propriedades de cache de um recurso para que possa decidir quando deve ou não utilizar cache.

4. Interface Uniforme e Baseado em Recursos: Uma interface uniforme entre cliente e servidor onde cada recurso é identificado por uma URI e são a entidade central no REST, recursos são manipulados por sua representação, mensagens auto-descritivas e HATEOAS. 
Recusos podem ser objetos/coleções de dados ou serviços.

5. Sitemas em camadas: O REST suporta a arquitetura em camadas, onde os componentes podem estar distribuidos em diferentes camadas, isso permite uma maior escalabilidade, separação de preocupações e flexibilidade na implementação. Podendo suportar load balancer, proxies, firewalls, etc.

# O que são Microserviços?

Microserviços são um estilo arquitetural de design de software em que uma aplicação é dividida em pequenos serviços independentes e autônomos. Cada serviço é responsável por uma função específica e pode ser desenvolvido, implantado e dimensionado de forma independente dos outros serviços.

Em contraste com uma abordagem monolítica, em que todas as funcionalidades são agrupadas em um único código-base e implantadas juntas, os microserviços permitem que os diferentes componentes da aplicação sejam desenvolvidos e mantidos separadamente.

Cada microserviço é construído em torno de um contexto de negócio delimitado, o que significa que ele se concentra em uma tarefa específica, como gerenciamento de usuários, processamento de pagamentos ou envio de e-mails. Os serviços podem se comunicar entre si por meio de APIs (Interfaces de Programação de Aplicativos) e trocar dados usando protocolos como HTTP/REST, mensagens assíncronas ou outros mecanismos.

Benefícios de microserviços incluem:

1. Escalabilidade e Flexibilidade: Esse estilo arquitetural permite escalar e implantar cada serviço individualmente, o que possiblita ajustar a capacidade e o desempenho de cada parte da aplicação de forma independente.

2. Manutenção Simplificada: As equipes de desenvolvimento podem trabalhar de forma mais independente, atualizando, corrigindo bugs ou adicionando novos recursos em serviços específicos sem afetar o restante da aplicação.

3. Resiliência e Tolerância a Falhas: Se um microserviço falha, os outros podem continuar funcionando normalmente, garantindo que a aplicação como um todo permaneça operacional.

4. Tecnologias e linguagens diversas: Cada microserviço pode ser implementado utilizando a tecnologia ou linguagem mais adequada para sua tarefa específica, permitindo utilizar a melhor ferramenta para cada necessidade.

No entanto, usar microserviços apresenta desafios como a complexidade entre serviços, a necessidade de gerenciar a consistência de dados distribuídos e a sobrecarga adicional na infraestrutura de implantação.

# Quais as desvantagens do cliente se comunicar diretamente com os microserviços?

Embora uma comunicação direta pareça ser uma abordagem eficiente e descentralizada, algumas das desvantagens são:

1. Acoplamento entre cliente e microserviço: Os clientes precisam saber os detalhes de implementação dos serviços, qualquer alteração nesses serviços pode exigir modificações significativas nos clientes. Isso pode levar a um cenário onde uma pequena alteração no serviço pode levar a várias modificações nos clientes, o que torna o sistema menos flexível e suscetível a falhas.

2. Segurança e Autorização: Quando o cliente se comunica diretamente com os microserviços, a gestão da segurança e autorização pode se tornar complexa, cada cliente precisa lidar com a autorização de cada serviço separadamente, isso pode levar a inconsistências na apllicação de politicas de segurança e tornar o sistema mais vulnerável a erros e violações de segurança. 

3. Escalabilidade limitada: A comunicação direta entre o cliente e os microserviços pode dificultar a escalabilidade horizontal. Se o número de clientes ou a carga de solicitações aumentar significativamente, pode ser necessário ajustar manualmente ou redistribuir as solicitações entre os microserviços para evitar gargalos.

# Como lidar com isso?

Uma forma é utilizar um API Gateway para intermediar a comunicação entre os clientes e os microserviços, o API Gateway pode lidar com a complexidade da comunicação, fornecer uma interface abstrata e simplificada para os clientes e gerenciar a autenticação, autorização e controle de acesso.

Com um API Gateway também é possível a coleta centralizada de métricas, logs e informações de monitoramento, versionamento de APIs, cache e load balancer.

# Como rodar o projeto?

Deixei um docker-compose para facilitar levantar o RabbitMQ e o MySQL. 

<img src="https://github.com/guilhermeLRibeiroo/microservices-dotnet6/assets/48655138/a5389999-40ae-4051-beff-662fc8ebc428"></img>

É necessário abrir a conexão com o MySQL para criar os bancos de dados. (eu utilizei HeidiSQL)

```
    IP: 127.0.0.1
    Porta: 3306
    User: root
    Password: rootpwd
```

E adicionar as seguintes databases:

+ shopping_product_api
+ shopping_order_api
+ shopping_email
+ shopping_coupon_api
+ shopping_cart_api
+ shopping_identity_server

E é para ficar assim:

<img src="https://github.com/guilhermeLRibeiroo/microservices-dotnet6/assets/48655138/bce40ecb-b22f-4ff0-8a16-5592c68756c4"></img>

Adicionei um código que aplica as migrations nos DbContexts uma única vez assim que iniciar o projeto.
### Não esquecer de selecionar "Multiple startup projects" na configuração de projeto inicial e definir como Start todos menos "DatabaseMigrations", "PaymentProcessor" e "MessageBus" pois são class libraries.

<img src="https://github.com/guilhermeLRibeiroo/microservices-dotnet6/assets/48655138/4be51f4d-7f96-408f-aaa7-ddfb7c2628ef"></img>

Depois disso é só dar Start no VisualStudio.
