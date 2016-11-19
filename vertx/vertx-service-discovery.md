### Service Discovery

#### Overview

이 컴포넌트는 서비스프록시, HTTP endpoint, Data Source같은 다양한 리소스를 찾고 배포할 수 있는 인프라를 제공합니다.
이 리소스들을 서비스(플랫폼에 종속되지 않는 표준 인터페이스를 통해서 기업의 업무를 표현한 Loosely coupled하고 상호 조합 가능한 소프트웨어 이다.)라고 부르겠습니다. 서비스는 발견 가능한 기능성입니다.
유형, 메타데이터, 위치에 따라 규정될 수 있습니다. 서비스는 데이터베이스, 서비스프록시, Http endpoint 다른 상상할 수 있는 것들이 될 수 있습니다. 
서비스는 vertx 엔티티일 필요는 없습니다. 각 서비스는 레코드로 설명됩니다. 

서비스 디스커버리는 service-orinted-computing 으로 정의된 인터랙션을 구현합니다. 어플리케이션은 서비스의 시작과 끝에 도달할 수 있습니다.(애매모호함)


- A service provider can:
 - publish a service record
 - un-publish a published record
 - update the status of a published service (down, out of service…​)

- A service consumer can:
 - lookup services
 - bind to a selected service (it gets a ServiceReference) and use it
 - release the service once the consumer is done with it
 - listen for arrival, departure and modification of services

- Consumer flow
 - 필요한 서비스레코드를 찾음
 - 서비스에 접근하기 위해 ServiceReference를 구함
 - 서비스에 접근하기 위한 서비스 오브젝트를 얻음
 - 종료 후 서비스 오브젝트를 반환함 

- 위에 flow를 보면 provider와 consumer가 공유하는 정보의 핵심은 records이다. 
- provider와 consumer는 자체 ServiceDiscovery 인스턴스를 생성해야 합니다. 
- 이 인스턴스들이 백그라운드(분산환경)에서 협력하여 일련의 서비스를 동기화 상태로 유지합니다.
- Service discovery는 다른 discovery기술(docker-swarm or kubernetes)들에서 service를 추가/삭제하기 위한 브리지를 제공합니다.


#### Using the service discovery
 - 3.3.3 버전 사용 (3.3.0에서 추가된 듯)

#### Overall concepts

- Service records
 - service-record는 서비스프로바이더에 의해 퍼블리쉬되는 서비스를 설명하는 객체이다. 
 - name, metadata, location(서비스가 어디있는지 설명하는) 정보를 가진다.
 - record는 provider와 consumer만 공유할 수 있다.
 - 메타 데이터 유형은 service type에 의해 포맷이 결정된다. 
 - record는 proivder가 사용할 수 있도록 준비되면 생성되고 provider가 멈추면 제거된다.

- Service Provider and publisher
 - service-provider는 service를 제공하는 엔티티
 - publisher는 recode(provider를 describe하는) 퍼블리싱하는 역할을 수행합니다.

- Service Consumer
 - service-consumer는 service discovery에서 service들을 찾습니다. 
 - 0 ~ n개의 record를 검색합니다. 
 - consumer는 이 records에서 ServiceReference(Consumer와 Provider를 바인딩할 수 있는)를 찾을 수 있습니다.
 - object 정리, service usage의 업데이트를 위해 service reference를 release하는 것은 중요하다. ???

- Service object
 - service에 접근할 수 있는 path를 가진 객체
 - 프록시나 클라이언트 같은 다양한 형태로 제공될 수 있다.
 - 서비스의 특성은 service-type 에 따라 다릅니다. (매우 추상적이다.)

- Service types
 - service는 단지 리소스다. 그리고 다양한 형태(database, restapi)로 존재할 수 있다.
 - vertx service discovery는 이런 이질성?을 처리할 수 있는 컨셉이 있다.
 - how the service is located (URI, event bus address, IP / DNS…​)
 - the nature of the service object (service proxy, HTTP client, message consumer…​)
 - 일부 서비스 유형은 service discovery component에 구현되어 있고 제공됩니다. 하지만 직접 구현할 수 있습니다.

- Service events
 - service provider는 publish와 withdrawn 이벤트를 가집니다. 
 - 이벤트는 이벤트버스에 의해 시작됩니다. 이벤트는 변경된 record를 가집니다. -> 이것도 결국 이벤트 버스구나
 - 추가로 어떤 reference가 검색되고 누가 사용하는 getReferece/release 메소드를 통해 알 수 있습니다. 

- Backend
 - service-discovery는 recode를 저장하기 위해 분산 스트럭처를 사용합니다.
 - default구현체에서는 클러스터의 모든 멤버는 이 recode에 접근할 수 있습니다.
 - ServiceDiscoveryBackend SPI를 직접 구현할 수도 있습니다.
 - discovery는 vertx clustering을 요구(다른 클러스터를 사용할 수 있단 말인듯)하진 않습니다. 
 - ServiceImporter를 통해 다른 클러스터 디스커버리 기술과 통합해서 사용할 수 있다.


#### Creating a service discovery instance
 - publisher와 consumer는 discovery infrastructure에서 사용하기위한 ServiceDiscovery 인스턴스를 직접 구현해야 한다.
 - default로 서비스 이벤트가 전송되는 이벤트 버스 주소는 vertx.discovery.announce 이다.
 - 또한 서비스 사용을 위한 이름을 지정할 수도 있다.
 - 더 이상 ServiceDiscovery 객체가 필요하지 않으면 close 해줘야 한다.

#### Publishing services
 - servicediscovery를 생성했다면 service를 publish할 수 있다.
 - Recode 클래스를 사용해 config를 구성하고 discovery를 통해 publish 한다.
 - recode도 처음부터 정의할 수 있고, 이미 지원하는 record 타입일 경우는 좀 더 수월하게 만들 수 있다.
 - recode는 registration id 기반으로 확장할 수 있으니 record 참조를 유지하는 것은 매우 중요하다.

#### Withdrawing services
 - service 내릴 땐 record id로 unpublish 한다.

#### Looking for service
 - 몇 번 반복되어 생략

#### Retrieving a service reference
 - Record를 선택하면 ServiceReference를 구할 수 있다. ServiceReference를 통해 service(e.g httpClient) 객체를 구할 수 있다.

#### Types of services

- 위에서 언급된 것처럼 service discovery는 다른 서비스들의 이질성을 관리하기 위해 service type 컨셉을 가지고 있습니다.
- default로 제공되는 type
 - HttpEndpoint : Rest API 용, url기반 
 - EventBusService : service proxy용, address 기반
 - MessageSource : publisher, address 기반
 - JDBCDataSource 

 - Services with no type
  - 타입이 없으므로 메타데이터를 보고 직접 구현해 사용한다.
 - Implementing your own service type
  - ServiceType SPI로 service type을 생성한다. (필요하면 봐야겠다; 너무 많다)

#### Listening for service arrivals and departures
- ServiceDiscoveryOptions에서 이벤트를 보내는 vertx.discovery.announce address를 설정할 수 있다.
- record는 상태 필드를 가진다.
 - UP, DOWN, OUT_OF_SERVICE

#### Service discovery bridges
- Docker, Kubernates, Consul같은 다른 discovery 메카니즘에 service를 추가/삭제하는를 지원한다.
- ServiceImporter 인터페이스를 구현하여 자체 브리지를 제공하고 registerServiceImporter를 사용하여 등록 할 수 있습니다.
- 말은 쉬운데... 구현예가 없어.. 아직 볼 이유는 없을듯

### 나의 생각

- 앞단에 partner -> service -> api 로 이어지는 호출권한 체크가 사라짐, 호출가능여부가 record에서 처리됨
- apigw에 * 이런 url처리가 사라짐, 웹서버가 white list 구현이 가능해짐
- 구현코드는 심플해지지만 구현난이도가 올라감, 구현체를 vertx내부에서 ServiceReference 형태로 주기 때문이다.
- ServiceReference의 변화가 일어나지 않는다면 계속 재사용이 가능하다. 참조관리도 내가 안하니 편할거 같다.
- apigw에서 서비스가 될 수 있는건 DB, Nbase, Portal, 백엔드 서버인 거 같음
- apigw에서 백엔드서버들을 서비스로 볼 수 있느냐, 없느냐가 관점인거 같음
- 설정정보의 패러다임이 변경되기 때문에 ... 굳이 안필요보이는데 ? 우린 api가 너무 많다... 심플한 apigw라면 괜찮은 거 같다. 
- 아 모르겠다.

### 참고삼아 SOA 

- Basic service (Description and Basic Operation)
 - service provider : (Publication, Capability, Interface, Behavior, QoS)
 - service aggregator(brocker) : Discovery, Selection, Binding, Composition
 - service clinent : 

- Composite service (Composition)
 - Coordination
 - Conformance
 - Monitoring
 - QoS

- Managed service
 - Operation
 - Market
