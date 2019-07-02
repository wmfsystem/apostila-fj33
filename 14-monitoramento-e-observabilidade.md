# Monitoramento e Observabilidade

## Exercício: expondo endpoints do Spring Boot Actuator

1. Adicione, a todos os projetos, uma dependência ao _starter_ do Spring Boot Actuator:

  ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  ```

  Essa dependência deve ser adicionada ao `pom.xml` do:

  - módulo `eats-application` do monólito
  - `eats-pagamento-service`
  - `eats-distancia-service`
  - `eats-nota-fiscal-service`
  - `api-gateway`
  - `service-registry`
  - `config-server`

2. Adicione, ao `application.properties` do `config-repo`, uma configuração para expôr todos os _endpoints_ do Actuator disponíveis:

  ####### config-repo/application.properties

  ```properties
  management.endpoints.web.exposure.include=*
  ```

  Não deixe de comitar a alteração:

  ```sh
  git commit -am "expõe endpoints do Spring Boot Actuator em todos os serviços"
  ```

  A configuração anterior será aplicada aos clientes do Config Server, que são os seguintes:

  - monólito
  - serviço de pagamentos
  - serviço de distância
  - serviço de nota fiscal
  - API Gateway

3. Para fazer com as requisições ao Actuator do API Gateway não acabem enviadas para o monólito, faça a configuração a seguir:

  ####### api-gateway/src/main/resources/application.properties

  ```properties
  zuul.routes.actuator.path=/actuator/**
  zuul.routes.actuator.url=forward:/actuator
  ```

4. Exponha todos os endpoints do Actuator no `config-server` e no `service-registry`:

  ####### config-server/src/main/resources/application.properties

  ```properties
  management.endpoints.web.exposure.include=*
  ```

  e

  ####### service-registry/src/main/resources/application.properties

  ```properties
  management.endpoints.web.exposure.include=*
  ```

5. Reinicie os serviços e explore os endpoints do Actuator.

  A seguinte URL contém links para os demais endpoints:

  http://localhost:{porta}/actuator

  É possível ver, de maneira detalhada, os valores das configurações:

  http://localhost:{porta}/actuator/configprops

  e

  http://localhost:{porta}/actuator/env

  Podemos verificar (e até modificar) os níveis de log:

  http://localhost:{porta}/actuator/loggers

  Com a URL a seguir, podemos ver uma lista de métricas disponíveis:

  http://localhost:{porta}/actuator/metrics

  Por exemplo, podemos obter o _uptime_ da JVM com a URL:

  http://localhost:{porta}/actuator/metrics/process.uptime

  Há uma lista dos `@RequestMapping` da aplicação:

  http://localhost:{porta}/actuator/mappings

  Podemos obter informações sobre os bindings, exchanges e channels do Spring Cloud Stream com as URLs:

  http://localhost:{porta}/actuator/bindings

  e

  http://localhost:{porta}/actuator/channels

  _Observação: troque `{porta}` pela porta de algum serviço._

  Há ainda endpoints específicos para o serviço que estamos acessando. Por exemplo, para o API Gateway temos com as rotas e _filters_:

  http://localhost:9999/actuator/routes

  e

  http://localhost:9999/actuator/filters

## Exercício: configurando o Hystrix Dashboard

1. Pelo navegador, abra `https://start.spring.io/`.
  Em _Project_, mantenha _Maven Project_.
  Em _Language_, mantenha _Java_.
  Em _Spring Boot_, mantenha a versão padrão.
  No trecho de _Project Metadata_, defina:

  - `br.com.caelum` em _Group_
  - `hystrix-dashboard` em _Artifact_

  Mantenha os valores em _More options_.

  Mantenha o _Packaging_ como `Jar`.
  Mantenha a _Java Version_ em `8`.

  Em _Dependencies_, adicione:

  - Hystrix Dashboard

  Clique em _Generate Project_.
2. Extraia o `hystrix-dashboard.zip` e copie a pasta para seu Desktop.
3. No Eclipse, no workspace de microservices, importe o projeto `hystrix-dashboard`, usando o menu _File > Import > Existing Maven Projects_.
4. Adicione a anotação `@EnableHystrixDashboard` à classe `HystrixDashboardApplication`:

  ####### hystrix-dashboard/src/main/java/br/com/caelum/hystrixdashboard/HystrixDashboardApplication.java

  ```java
  @EnableHystrixDashboard
  @SpringBootApplication
  public class HystrixDashboardApplication {

    public static void main(String[] args) {
      SpringApplication.run(HystrixDashboardApplication.class, args);
    }

  }
  ```

  Adicione o import:

  ```java
  import org.springframework.cloud.netflix.hystrix.dashboard.EnableHystrixDashboard;
  ```

5. No arquivo `application.properties`, modifique a porta para `7777`:

  ####### hystrix-dashboard/src/main/resources/application.properties

  ```properties
  server.port=7777
  ```

6. Execute a classe `HystrixDashboardApplication`.

7. Acesse o Hystrix Dashboard, pelo navegador, com a seguinte URL:

  http://localhost:7777/hystrix

  Coloque, na URL, o endpoint de Hystrix Stream Actuator do API Gateway:

  http://localhost:9999/actuator/hystrix.stream

  Clique em _Monitor Stream_.

  Em outra aba, acesse URLs do API Gateway como as que seguem:

  - http://localhost:9999/restaurantes/1, que exibirá o circuit breaker do monolito
  - http://localhost:9999/pagamentos/1, que exibirá o circuit breaker do serviço de pagamentos
  - http://localhost:9999/distancia/restaurantes/mais-proximos/71503510, que exibirá o circuit breaker do serviço de distância
  - http://localhost:9999/restaurantes-com-distancia/71503510/restaurante/1, que exibirá os circuit breakers relacionados a composição de chamadas feita no API Gateway

## Exercício: agregando dados dos circuit-breakers com Turbine

1. Pelo navegador, abra `https://start.spring.io/`.
  Em _Project_, mantenha _Maven Project_.
  Em _Language_, mantenha _Java_.
  Em _Spring Boot_, mantenha a versão padrão.
  No trecho de _Project Metadata_, defina:

  - `br.com.caelum` em _Group_
  - `turbine` em _Artifact_

  Mantenha os valores em _More options_.

  Mantenha o _Packaging_ como `Jar`.
  Mantenha a _Java Version_ em `8`.

  Em _Dependencies_, adicione:

  - Turbine
  - Eureka Client
  - Config Client

  Clique em _Generate Project_.
2. Extraia o `turbine.zip` e copie a pasta para seu Desktop.
3. No Eclipse, no workspace de microservices, importe o projeto `turbine`, usando o menu _File > Import > Existing Maven Projects_.
4. Adicione as anotações `@EnableDiscoveryClient` e `@EnableTurbine` à classe `TurbineApplication`:

  ####### turbine/src/main/java/br/com/caelum/turbine/TurbineApplication.java

  ```java
  @EnableTurbine
  @EnableDiscoveryClient
  @SpringBootApplication
  public class TurbineApplication {

    public static void main(String[] args) {
      SpringApplication.run(TurbineApplication.class, args);
    }

  }
  ```

  Não esqueça de ajustar os imports:

  ```java
  import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
  import org.springframework.cloud.netflix.turbine.EnableTurbine;
  ```

5. No arquivo `application.properties`, modifique a porta para `7776`.

  Adicione configurações que aponta para os nomes das aplicações e para o cluster `default`:

  ####### turbine/src/main/resources/application.properties

  ```properties
  server.port=7776

  turbine.appConfig=apigateway
  turbine.clusterNameExpression='default'
  ```

6. Defina um arquivo `bootstrap.properties` no diretório de _resources_, configurando o endereço do Config Server:

  ```properties
  spring.application.name=turbine
  spring.cloud.config.uri=http://localhost:8888
  ```

7. Execute a classe `TurbineApplication`.

  Então, acesse o Turbina pela URL a seguir:

  http://localhost:7776/turbine.stream

  Em outra janela do navegador, faça algumas chamadas ao API Gateway, como as do exercício anterior.

  Observe, na página do Turbine, um fluxo de dados parecido com:

  ```txt
  : ping
  data: {"reportingHostsLast10Seconds":0,"name":"meta","type":"meta","timestamp":1562070789955}

  : ping
  data: {"reportingHostsLast10Seconds":0,"name":"meta","type":"meta","timestamp":1562070792956}

  : ping
  data: {"rollingCountFallbackSuccess":0,"rollingCountFallbackFailure":0,"propertyValue_circuitBreakerRequestVolumeThreshold":20,"propertyValue_circuitBreakerForceOpen":false,"propertyValue_metricsRollingStatisticalWindowInMilliseconds":10000,"latencyTotal_mean":0,"rollingMaxConcurrentExecutionCount":0,"type":"HystrixCommand","rollingCountResponsesFromCache":0,"rollingCountBadRequests":0,"rollingCountTimeout":0,"propertyValue_executionIsolationStrategy":"THREAD","rollingCountFailure":0,"rollingCountExceptionsThrown":0,"rollingCountFallbackMissing":0,"threadPool":"monolito","latencyExecute_mean":0,"isCircuitBreakerOpen":false,"errorCount":0,"rollingCountSemaphoreRejected":0,"group":"monolito","latencyTotal":{"0":0,"99":0,"100":0,"25":0,"90":0,"50":0,"95":0,"99.5":0,"75":0},"requestCount":0,"rollingCountCollapsedRequests":0,"rollingCountShortCircuited":0,"propertyValue_circuitBreakerSleepWindowInMilliseconds":5000,"latencyExecute":{"0":0,"99":0,"100":0,"25":0,"90":0,"50":0,"95":0,"99.5":0,"75":0},"rollingCountEmit":0,"currentConcurrentExecutionCount":1,"propertyValue_executionIsolationSemaphoreMaxConcurrentRequests":10,"errorPercentage":0,"rollingCountThreadPoolRejected":0,"propertyValue_circuitBreakerEnabled":true,"propertyValue_executionIsolationThreadInterruptOnTimeout":true,"propertyValue_requestCacheEnabled":true,"rollingCountFallbackRejection":0,"propertyValue_requestLogEnabled":true,"rollingCountFallbackEmit":0,"rollingCountSuccess":0,"propertyValue_fallbackIsolationSemaphoreMaxConcurrentRequests":10,"propertyValue_circuitBreakerErrorThresholdPercentage":50,"propertyValue_circuitBreakerForceClosed":false,"name":"RestauranteRestClient#porId(Long)","reportingHosts":1,"propertyValue_executionIsolationThreadPoolKeyOverride":"null","propertyValue_executionIsolationThreadTimeoutInMilliseconds":1000,"propertyValue_executionTimeoutInMilliseconds":1000}
  ```

8. Vá novamente ao Hystrix Dashboard, pela URL:

  http://localhost:7777/hystrix

  Na URL, use o endereço da stream do Turbine:

  http://localhost:7776/turbine.stream

  Faça algumas chamadas ao API Gateway, como nos passos anteriores.

  Depois disso, veja os status dos circuit breakers.

## Exercício: agregando baseado em eventos com Turbine Stream

1. No `pom.xml` do projeto `turbine`, troque a dependência ao starter do Turbine pela do Turbine Stream. Adicione também o binder do Spring Cloud Stream ao RabbitMQ:

  ####### turbine/pom.xml

  ```xml
  <̶d̶e̶p̶e̶n̶d̶e̶n̶c̶y̶>̶
    <̶g̶r̶o̶u̶p̶I̶d̶>̶o̶r̶g̶.̶s̶p̶r̶i̶n̶g̶f̶r̶a̶m̶e̶w̶o̶r̶k̶.̶c̶l̶o̶u̶d̶<̶/̶g̶r̶o̶u̶p̶I̶d̶>̶
    <̶a̶r̶t̶i̶f̶a̶c̶t̶I̶d̶>̶s̶p̶r̶i̶n̶g̶-̶c̶l̶o̶u̶d̶-̶s̶t̶a̶r̶t̶e̶r̶-̶n̶e̶t̶f̶l̶i̶x̶-̶t̶u̶r̶b̶i̶n̶e̶<̶/̶a̶r̶t̶i̶f̶a̶c̶t̶I̶d̶>̶
  <̶/̶d̶e̶p̶e̶n̶d̶e̶n̶c̶y̶>̶
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-turbine-stream</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
  </dependency>
  ```

2. O status de cada circuit breaker será obtido por meio de eventos no Exchange `springCloudHystrixStream` do RabbitMQ.

  Por isso, remova as configurações de aplicações do `application.properties`:

  ####### turbine/src/main/resources/application.properties

  ```properties
  t̶u̶r̶b̶i̶n̶e̶.̶a̶p̶p̶C̶o̶n̶f̶i̶g̶=̶a̶p̶i̶g̶a̶t̶e̶w̶a̶y̶
  t̶u̶r̶b̶i̶n̶e̶.̶c̶l̶u̶s̶t̶e̶r̶N̶a̶m̶e̶E̶x̶p̶r̶e̶s̶s̶i̶o̶n̶=̶'̶d̶e̶f̶a̶u̶l̶t̶'̶
  ```

3. Troque a anotação `@EnableTurbine` por `@EnableTurbineStream` na classe `TurbineApplication`:

  ####### turbine/src/main/java/br/com/caelum/turbine/TurbineApplication.java

  ```java
  @̶E̶n̶a̶b̶l̶e̶T̶u̶r̶b̶i̶n̶e̶
  @EnableTurbineStream
  @EnableDiscoveryClient
  @SpringBootApplication
  public class TurbineApplication {

    public static void main(String[] args) {
      SpringApplication.run(TurbineApplication.class, args);
    }

  }
  ```

  Ajuste os imports da seguinte maneira:

  ```java
  i̶m̶p̶o̶r̶t̶ ̶o̶r̶g̶.̶s̶p̶r̶i̶n̶g̶f̶r̶a̶m̶e̶w̶o̶r̶k̶.̶c̶l̶o̶u̶d̶.̶n̶e̶t̶f̶l̶i̶x̶.̶t̶u̶r̶b̶i̶n̶e̶.̶E̶n̶a̶b̶l̶e̶T̶u̶r̶b̶i̶n̶e̶;̶
  import org.springframework.cloud.netflix.turbine.stream.EnableTurbineStream;
  ```

4. Adicione a dependência ao Hystrix Stream no `pom.xml` do API Gateway:

  ####### api-gateway/pom.xml

  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-hystrix-stream</artifactId>
  </dependency>
  ```

5. Ajuste o _destination_ do _channel_ `hystrixStreamOutput`, no `application.properties` do API Gateway, por meio da propriedade:

  ####### api-gateway/src/main/resources/application.properties

  ```properties
  spring.cloud.stream.bindings.hystrixStreamOutput.destination=springCloudHystrixStream
  ```

6. Reinicie o API Gateway e o Turbine.

  Acesse, pelo navegador, o Hystrix Dashboard:

  http://localhost:7777/hystrix

  Aponte para a URL do Turbine:

  http://localhost:7776/turbine.stream

  Chame o API Gateway em outra janela do navegador, como as dos exercícios anteriores.

  Observe informações sobre os circuit breakers no Hystrix Dashboard.

## Exercício: configurando o Zipkin no Docker Compose

1. Para provisionar uma instância do Zipkin, adicione as seguintes configurações ao `docker-compose.yml` do seu Desktop:

  ####### docker-compose.yml

  ```yml
  zipkin:
    image: openzipkin/zipkin
    ports:
      - "9410:9410"
      - "9411:9411"
    depends_on:
      - rabbitmq
    environment:
      RABBIT_URI: "amqp://eats:caelum123@rabbitmq:5672"
  ```

2. Execute o servidor do Zipkin pelo Docker Compose com o comando:

  ```sh
  docker-compose up
  ```

3. Acesse a UI Web do Zipkin pelo navegador através da URL:

  http://localhost:9411/zipkin/

## Exercício: enviando informações para o Zipkin com Spring Cloud Sleuth

1. Adicione uma dependência ao starter do Spring Cloud Zipkin no `pom.xml` do API Gateway:

  ####### api-gateway/pom.xml

  ```xml
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
  </dependency>
  ```

  Faça o mesmo no `pom.xml` do:

  - módulo `eats-application` do monólito
  - serviço de pagamentos
  - serviço de distância
  - serviço de nota fiscal

2. Por padrão, o Spring Cloud Sleuth faz rastreamento por amostragem de 10% das chamadas. É um bom valor, mas inviável pelo pouco volume de nossos requests.

  Por isso, altere a porcentagem de amostragem para 100%, modificando o arquivo `application.properties` do `config-repo`:

  ####### config-repo/application.properties

  ```properties
  spring.sleuth.sampler.probability=1.0
  ```

  Não esqueça de comitar a alteração:

  ```sh
  git commit -m "configuração do Spring Cloud Sleuth"
  ```

3. Reinicie os serviços que foram modificados no passo anterior. Garanta que a UI esteja no ar.

  Faça um novo pedido, até a confirmação do pagamento. Faça o login como dono do restaurante e aprove o pedido. Edite o tipo de cozinha e/ou CEP de um restaurante.

  Vá até a interface Web do Zipkin e, em selecione um serviço em _Service Name_. Então, clique em _Find traces_ e veja os rastreamentos. Clique para ver os detalhes.

  Na aba _Dependencies_, veja um gráfico com as dependências entre os serviços baseadas no uso real (e não apenas em diagramas arquiteturais).

## Exercício: Spring Boot Admin

1. Pelo navegador, abra `https://start.spring.io/`.
  Em _Project_, mantenha _Maven Project_.
  Em _Language_, mantenha _Java_.
  Em _Spring Boot_, mantenha a versão padrão.
  No trecho de _Project Metadata_, defina:

  - `br.com.caelum` em _Group_
  - `admin-server` em _Artifact_

  Mantenha os valores em _More options_.
 
  Mantenha o _Packaging_ como `Jar`.
  Mantenha a _Java Version_ em `8`.

  Em _Dependencies_, adicione:

  - Spring Boot Admin (Server)
  - Config Client
  - Eureka Discovery Client

  Clique em _Generate Project_.
2. Extraia o `admin-server.zip` e copie a pasta para seu Desktop.
3. No Eclipse, no workspace de microservices, importe o projeto `admin-server`, usando o menu _File > Import > Existing Maven Projects_.
4. Adicione a anotação `@EnableAdminServer` à classe `AdminServerApplication`:

  ####### admin-server/src/main/java/br/com/caelum/adminserver/AdminServerApplication.java

  ```java
  @EnableAdminServer
  @SpringBootApplication
  public class AdminServerApplication {

    public static void main(String[] args) {
      SpringApplication.run(AdminServerApplication.class, args);
    }

  }
  ```

  Adicione o import:

  ```java
  import de.codecentric.boot.admin.server.config.EnableAdminServer;
  ```

5. No arquivo `application.properties`, modifique a porta para `6666`:

  ####### admin-server/src/main/resources/application.properties

  ```properties
  server.port=8084
  ```

6. Crie um arquivo `bootstrap.properties` no diretório `src/main/resources` do Admin Server, definindo o nome da aplicação e o endereço do Config Server:

  ```properties
  spring.application.name=adminserver

  spring.cloud.config.uri=http://localhost:8888
  ```

7. Execute a classe `AdminServerApplication`.

  Pelo navegador, acesse a URL:

  http://localhost:8084

  Veja informações sobre as aplicações e instâncias.

  Em _Wallboard_, há uma visualização interessante do status dos serviços.