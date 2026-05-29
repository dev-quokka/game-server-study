# MongoDB

MongoDB는 Document 구조로 데이터를 저장하는  
오픈소스 기반의 NoSQL 데이터베이스입니다.
RDBMS와 달리 정해진 스키마 없이  
JSON과 유사한 BSON 형식으로 데이터를 저장합니다.

<br>

## MongoDB는 왜 필요한가?

RDBMS는 정해진 스키마와 테이블 구조를 따르기 때문에  
데이터 구조 변경이 잦은 환경에서는 유연성이 떨어집니다.
또한 대용량 데이터 처리 시  
수직 확장(Scale-up)에 한계가 존재합니다.
특히 유저 데이터 구조가 자주 변하는 게임 서버에서는  
컬럼 추가/변경 비용이 큰 RDBMS가 부담이 될 수 있습니다.

> 이를 해결하기 위해 **NoSQL 데이터베이스**를 사용하며  
> 대표적인 Document DB가 MongoDB입니다.  
> Document: Key-Value 쌍의 집합으로 이루어진 데이터 단위 (JSON과 유사한 BSON 형식)

<br>

## 데이터 구조

### Document & Collection

- **Document**: RDBMS의 Row에 해당, BSON 형식의 데이터 단위
- **Collection**: RDBMS의 Table에 해당, Document들의 집합
- **Database**: Collection들의 집합

<br>

### Embedded Document

- Document 안에 또 다른 Document를 중첩 저장
- JOIN 없이 관련 데이터를 한 번에 조회 가능
  > 읽기 성능이 좋지만  
  > **중복 데이터 발생 가능성이 있어 설계 시 주의**

<br>

## 특징

- Document(BSON) 구조
- Schema-less → 유연한 데이터 구조
- 다양한 데이터 타입 지원  
  (String, Number, Array, Object, Date, ObjectId 등)
- 인덱싱 지원  
  → 단일/복합/Geo/Text 인덱스 등 다양한 방식 제공
- 수평 확장(Sharding) 기반 처리  
  → 대용량 데이터에 효율적

<br>

## 장점

#### 1. 유연한 스키마

- 스키마 변경 없이 필드 추가/삭제 가능
- 빠르게 변하는 요구사항에 대응 용이

#### 2. 수평 확장성

- Sharding을 통한 데이터 분산 저장
- 대용량 트래픽 처리에 강점

#### 3. 직관적인 데이터 모델

- JSON 형태로 객체와 매핑이 자연스러움
- ORM/ODM 사용 시 개발 생산성 향상

#### 4. 고가용성

- Replica Set 지원
- Primary 장애 시 자동 Failover

<br>

## 단점

#### 1. 트랜잭션 제약

- 4.0부터 멀티 Document 트랜잭션 지원하지만  
  → RDBMS 대비 성능 비용이 크고 제약 존재

#### 2. JOIN 연산의 한계

- lookup으로 JOIN 유사 연산 가능하지만  
  → RDBMS만큼 효율적이지 않음
- 복잡한 관계형 데이터 처리에 부적합

#### 3. 메모리 사용량

- 인덱스와 Working Set을 메모리에 유지  
  → 메모리 사용량이 큰 편

#### 4. 데이터 중복

- Embedded 구조 사용 시 데이터 중복 발생  
  → 일관성 관리 비용 증가

<br>

## 게임 서버에서의 활용

- 유저 프로필 / 계정 정보
- 인벤토리 데이터 (구조가 자주 변하는 아이템)
- 게임 로그 / 플레이 기록
- 매치 히스토리
- 우편함 / 알림 데이터
- 길드, 친구 관계 등 소셜 데이터

<br>

## 📌 핵심 정리

- MongoDB는 **유연한 스키마와 확장성을 위한 Document DB**
- 변화가 잦은 데이터 구조와 대용량 처리에 강점
- 강한 정합성이 필요한 데이터에는 RDBMS와 함께 사용

<br>

> 게임 서버에서는  
> **유저 데이터 + 로그 + 소셜 데이터** 관리에 핵심적으로 사용됩니다.
