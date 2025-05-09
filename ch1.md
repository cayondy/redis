# 1장 마이크로서비스 아키텍쳐와 레디스


## 모놀리틱 아키텍처 vs 마이크로서비스 아키텍처

## NoSQL이란?

- No SQL 혹은 Not Only SQL
- 관계가 정의되어 있지 않은 데이터를 저장한다.

  - 특징
    - 실시간 응답 : 빠른 응답 속도
    - 확장성 : 비정기적인 이벤트에도 유연한 확장
    - 고가용성 : 빠른 장애 복구
    - 클라우드 네이티브 
    - 단순성 : 서비스별 적절한 데이터 모델 사용이 가능
    - 유연성 : 다양한 데이터 유형을 적합한 형식으로 저장 가능


## NoSQL 데이터 저장소 유형

- 그래프 유형
  - 엔티티 간의 관계를 효율적으로 저장
  - 노드, 엣지, 속성 으로 데이터를 나타낸다.<br/>
  ```
  노드 = 엔티티, 엣지 = 데이터의 관계<br/>
  에지는 시작 노드, 끝 노드, 유형과 방향을 가짐
  ```
  - 관계를 저장하고 표현할 때 유용하게 사용
  - 속성의 크기가 크거나 속성의 갯수가 많을 때는 적절하지 않다.
 

- 칼럼 유형
  - 테이블을 행이 아닌 열을 기준으로 저장
  - 데이터는 하나의 열에 중첩된 키-값 형태로 저장 가능 -> 유연한 스키마 저장
  - 대량의 데이터에 대한 집계 쿼리의 빠른 처리 -> 빅데이터 처리에 적합
 

- 문서 유형
  - JSON 형태로 데이터를 저장 -> 효율적이고 직관적으로 데이터를 사용할 수 있다.
  - 스키마가 정해져 있지 않아서 유연하게 데이터를 저장할 수 있다.
  - 모든 값은 키와 연결되는 계층적 트리 구조를 갖는다.
  - 데이터를 저장, 검색하는데 효과적


- 키-값 유형
  - 가장 단순하고 빠른 유형
  - 모든 값이 키와 연결되고 키도 유의미한 데이터이다.
  - 값은 키에 종속되어 있다.
  - 구조가 단순하기 때문에 데이터 접근과 처리가 빠르다. -> 실시간 서비스에 적합
 


## 레디스란?

- Remote dictionary server
- 고성능의 키-값 유형의 인메모리 NoSQL 데이터베이스

  - 특징
    1. 인메모리 형태의 데이터베이스로 데이터 처리 성능이 빠르다.
    2. 키-값 형태로 데이터를 관리하고 키에 다양한 형태의 데이터 타입 매핑이 가능하다. -> 추가적인 데이터 가공이 필요없음
      ```
      - 임피던스 불일치
      관계형 데이터베이스의 테이블과 프로그래밍 언어 간 데이터 구조, 기능 차이로 인해 발생하는 충돌
      -> 레디스는 다양한 자료 구조를 지원하므로 개발에 편리하다.
      ```
    3. 다양한 오픈 소스 클라이언트 사용이 가능하고 다양한 언어를 지원한다.
    4. 이벤트 루프를 이용한 싱글 스레드로 작동한다.
      ```
      정확히는 하나의 메인 스레드와 3개의 별도 스레드로 구성
      오래 걸리는 특정 작업을 수행하면 인적 장애 발생 가능성이 있다!
      ```
    5. 자체적인 고가용성, HA(high Availability) 기능을 제공
      ```
      복제를 통해 데이터를 여러 서버에 분산 가능
      센티널을 활용해 장애 탐지 시 자동으로 페일오버 시켜준다.
      - 페일오버 : 장애가 발생한 Redis 마스터를 슬레이브로 대체하는 과정
      ```
    6. 클러스터 모드 사용 시, 수평적 확장이 용이하다.
      ```
      - 클러스터 모드
      마스터, 슬레이브 노드로 구성
      클러스터 버스라는 프로토콜로 서로 감시가 가능
      페일오버 기능 제공
      ```

      | 항목              | Redis Sentinel              | Redis Cluster           |
      | --------------- | --------------------------- | ----------------------- |
      | **데이터 분산**      | ❌ 단일 마스터, 분산 불가             | ✅ 16384 슬롯 기반 분산 저장     |
      | **수평 확장성**      | ❌ 수직 확장만 가능                 | ✅ 노드 수 추가로 확장 가능        |
      | **자동 Failover** | ✅ 지원                        | ✅ 지원                    |
      | **클라이언트 인식**    | ❌ 클라이언트는 Sentinel 통해 마스터 확인 | ✅ 클라이언트가 직접 해시 슬롯 인식 필요 |
      | **복잡성**         | ✅ 상대적으로 단순                  | ❌ 구성 및 운영 복잡함           |

     7. 클라우드 네이티브 방식으로 클라우드 환경에 특화되어 있다.<br/>
        여러 클라우드 제공업체를 혼합해서 활용하는 멀티 클라우드 환경도 가능



## 마이크로서비스 아키텍처와 레디스

- 데이터 저장소로서의 레디스
  - 마이크로서비스 아키텍처에서 각 서비스 별 개별 저장소로 사용하기 좋음
  - 간편한 설치
  - 최소한의 리소스로 많은 처리량에 대응 가능
  - 다양한 자료 구조 제공 + 편리한 사용
  - 메모리에 있는 데이터가 영구 저장되지 않음<br/>
    -> AOF(Append Only File), RDB(Redis DataBase) 형식으로 디스크에 저장 가능
 
- 메시지 브로커로서의 레디스
  ```
  - 마이크로서비스 아키텍처에서 각 서비스는 완전히 분리되어 있기 때문에 다른 서비스간 지속적인 통신이 필요하다.
  - 메시징 큐 혹은 stream 과 같은 메세지 브로커를 이용해 비동기 통신 채널을 구현하는 것을 권장
  ```
  - stream 자료 구조를 이용해 스트림 플랫폼으로 사용이 가능
  ```
  - stream
  아파치 카프카 시스템에서 영감을 받아 만들어진 자료 구조
  데이터를 계속 추가하는 방식으로 저장
  데이터를 읽을 수 있는 소비자와 그룹을 관리 -> 데이터 분산 처리 및 검색이 가능
  ```


