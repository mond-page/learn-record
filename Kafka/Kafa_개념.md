### Topic

- 데이터가 들어갈 수 있는 공간
- Topic을 여러개를 생성 할 수 있음
- 하나의 토픽은 여러개의 파티션으로 구성 될 수 있음
- 내부에 데이터가 큐처럼 쌓이게 되고, 컨슈머는 오래된 순서대로 가져가게 된다
    - 컨슈머가 record들을 가져가더라도 데이터는 삭제되지 않음
    - 새로운 컨슈머가 붙었을 때 다시 가져갈 수 있음 (컨슈머 그룹이 다르고, auto.offset.reset = earliset)
- 키가 NULL 일 경우 Round Robin으로 여러 파티션으로 들어가게 됨
    - 파티션 늘리는 건 가능하지만, 줄이는 건 불가능하기 때문에 조심
    - Partition의 record는 언제 삭제될까? → 옵션에 따라 다름

### Broker, Replication, ISR

- Broker
    - 카프카가 설치되어 있는 서버 단위
        - 보통 3개 이상으로 권유

- Replication
    - partition의 복제를 의미
    - partition의 고가용성을 위해 존재
        - 브로커가 갑자기 문제가 된다면 다른 복제본을 얻을 수 있음
    - producer의 ack 옵션은 0, 1, all
        - `0` → Leader partition의 데이터를 전송하고 응답값을 받지 않음 / 나머지 파티션에 복제되었는지 알 수 없음
            - 속도는 빠르지만, 데이터 유실 가능성 존재
        - `1` → Leader partition의 데이터 전송하고 응답값을 받지 않음 / 나머지 파티션에 복제되었는지 알 수 없음
            - Leader partition이 받은 즉시 장애가 난다면 나머지 파티션에 데이터가 미쳐 전송되지 못한 상태
            - 데이터 유실 가능성
        - `all` → follwer partition에 복제가 잘 이루어졌는지 응답값을 받음
            - 나머지 Follwer partition에도 저장되었는지 확인함
            - 데이터 유실은 없지만, 확인하는 부분이 많아서 속도가 느림
- ISR(In Sync Replica)
    - Leader partition / Follower partition 를 합친 의미

### Partitioner

- Producer의 중요 개념 중 하나
- 데이터를 토픽에 어떤 파티션에 넣을지 결정하는 역할
- 프로듀서를 사용 할 때 기본 파티셔너는 `UniformStickyPartitioner`
    - 메세지 키가 있는 경우
        - 특정한 hash 값으로 생성 → 어느 파티션에 들어갈지 정해지게 됨
    - 메세지 키가 없는 경우
        - 라운드 로빈 방식으로 들어가게 됨
        - 배치로 모을 수 있는 최대한으로 모아 파티션으로 보냄
- 커스텀 파티셔너도 설정해서 사용 할 수 있음
    - Partitioner Interface를 구현해서 사용

### Consumer Lag

- Producer가 마지막으로 넣은 offset과 Consumer가 마지막으로 읽은 offset가의 차이를 의미
- **주로 컨슈머의 상태를 볼 때 사용**
- 여러개의 파티션이 존재하게 된다면 Lag이 여러개가 될 수 있음
    - 그 중 높은 숫자의 lag를 `records-lag-max`

### 카프카 Burrow

- 오픈소스 기반의 Consumer Lag 모니터링 애플리케이션
- kafkaConsumer 객체를 통해 현재 lag 정보를 가져올 수 있음
    - Consumer 로직단에서 lag 수집시 컨슈머 상태에 디펜던시 발생
    - 비정상적으로 종료되면 Lag 정보를 보낼 수 없음
- Burrow 특징
    - 멀티 카프카 클러스터 지원
    - Sliding Window를 통한 Consumer의 Status 확인
        - `ERROR`, `WARNING`, `OK`
    - HTTP api 제공

### Kafka, RabbitMQ, Redis Queue 차이점

- 메세지 브로커
    - 이벤트 브로커 역할을 할 수 없다
    - 메세지를 받아 적절히 처리하고 나면 즉시 또는 짧은 시간 내에 삭제되는 구조
    - 레디스 큐, 레빗엠 큐

- 이벤트 브로커
    - 메세지 브로커 역할을 할 수 있다
    - 두가지 특징
        - 이벤트 또는 메세지라고 불리는 레코드를 딱 하나만 보관하고 인덱스를 통해 개별 엑세스 관리
        - 업무상 필요한 시간동안 이벤트 보존
    - 서비스에서 나오는 이벤트를 큐에 저장
        - 딱 한 번 일어난 이벤트를 브로커에 저장하면서 단일 진실 공급원으로 사용
        - 장애 발생시, 장애가 일어난 시점부터 재처리 가능
        - 많은 양의 실시간 스트림 데이터를 효과적으로 처리
    - 카프카, AWS 키네시스
