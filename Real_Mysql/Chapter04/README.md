# 아키텍처
* MySQL 서버 -> 사람의 머리 역할을 한다.
* 스토리지 엔진 -> 엔진과 손발 역할을 한다.
  * `스토리지 엔진`: 데이터를 실제로 저장하고 읽는 역할을 한다.
    * MySQL은 여러 종류의 스토리지 엔진을 지원한다.
        * 대표적인 스토리지 엔진: InnoDB, MyISAM
        * `InnoDB`: 트랜잭션을 지원하고, 외래 키 제약 조건을 지원한다. (MySQL의 기본 스토리지 엔진)
        * `MyISAM`: 트랜잭션과 외래 키 제약 조건을 지원하지 않는다. (읽기 성능이 더 좋다.)
> 트랜잭션: 하나의 작업 단위를 의미 여러 개의 SQL 문장을 논리적으로 하나로 묶어서 처리

## 4.1 MySQL 엔진 아키텍처
### 4.1.1 MySQL 전체 구조
![아키텍처](https://velog.velcdn.com/images/bbora/post/12f21fc9-6e9a-4de6-81d7-59a76f6c74e7/image.png)
* MySQL은 일반 상용 RDBMS와 같이 대부분의 프로그래밍 언어로부터 접근 방법을 지원한다.
* MySQL 서버는 크게 MySQL 엔진과 스토리지 엔진으로 나눌 수 있다.
  * 이 둘을 합쳐 MySQL 혹은 MySQL 서버라고 한다.

### 4.1.1.1 MySQL 엔진
* MySQL 엔진은 커넥션 핸들러와 SQL 파서 및 전처리기, 옵티마이저가 중심을 이룬다.
  * 커넥션 핸들러: 클라이언트로부터 접속 요청을 받아들이고, 접속을 관리하는 역할
  * SQL 파서: SQL 문법을 분석하고 이해하는 역할
    * 문법 체크 → SQL 문법이 맞는지 확인 (SELECT * FORM users → 에러)
  * 전처리기: 실행 계획을 수립하는 역할
    * 테이블과 칼럼 존재 여부 확인
    * 사용자 권한 확인
    * 파라미터 바인딩 -> select * from users where id = ? (전처리기가 ?에 값을 바인딩)
  * 옵티마이저: 쿼리의 최적화된 실행을 위한 역할
* MySQL은 표준 SQL 문법을 지원하기 때문에 타 DBMS에서 작성한 SQL 문장을 그대로 사용할 수 있다.

### 4.1.1.2 스토리지 엔진
* MySQL 엔진 -> 요청된 SQL 문장을 분석 최적화하는 역할
* 스토리지 엔진 -> 실제 데이터를 디스크 스토리지에 저장하거나 디스크 스토리지로부터 데이터를 읽어오는 역할
* MySQL 엔진은 하나지만 스토리지 엔진은 여러개를 동시에 사용 할 수 있다.
  * 다음과 같이 테이블이 사용할 스토리지 엔진을 지정하면 이후 해당 테이블의 모든 읽기 작업이나 변경 작업은 정의된 스토리지 엔진이 처리한다.
  ```sql
    CREATE TABLE test_table (fd1 INT, fd2 INT) ENGINE = InnoDB; -- InnoDB 스토리지 엔진을 사용(기본값)
    ```
* 각 스토리지 엔진은 성능 향상을 위해 키 캐시(MyISAM 스토리지 엔진)나 InnoDB 버퍼풀(InnoDB 스토리지 엔진)과 같은 기능을 내장하고 있다.
  * 키 캐시: MyISAM 테이블의 인덱스(Index)를 메모리에 올려두고 빠르게 접근하도록 하는 기능
    * 즉 디스크에서 매번 읽지 않고 메모리에서 바로 읽어서 성능 향상
  * InnoDB 버퍼풀: InnoDB 스토리지 엔진이 사용하는 메모리 공간으로, 데이터와 인덱스를 캐싱하여 디스크 I/O를 줄이고 성능을 향상시키는 역할
### 4.1.1.3 핸들러 API
* MySQL 엔진의 쿼리 실행기에서 데이터를 쓰거나 읽어야 할 때는 각 스토리지 엔진에 쓰기 또는 읽기를 요청하는데, 이러한 요청을 `핸들러 요청`이라 한다.
  * 여기서 사용되는 API를 `핸들러 API`라 한다.
* InnoDB 스토리지 엔진 또한 이 핸들러 API를 이용해 MySQL 엔진과 데이터를 주고 받는다.
### 4.1.2 MySQL 스레딩 구조
![스레딩 구조](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FPHGWg%2FbtrBNcXzAis%2FAAAAAAAAAAAAAAAAAAAAAIudmCSR4xewMHC-mssZZX4U8YMaQ2Q9uaYydZmrcdeO%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1759244399%26allow_ip%3D%26allow_referer%3D%26signature%3DKPgSe0V%252BA8%252FFVF6rA9ft8nWAOLw%253D)

* MySQL 서버는 `프로세스` 기반이 아니라 `스레드` 기반으로 작동한다.
  * 프로세스: 운영체제에서 독립된 실행 단위 서로 메모리 공간을 공유하지 않고 새로운 프로세스를 만들면 상대적으로 무겁다.
  * 스레드: 프로세스 안에서 동작하는 가벼운 실행 단위 같은 프로세스 안에서 메모리 공간을 공유 하기 때문에 스레드를 만드는 비용이 훨씬 적다.

#### **MySQL 스레드 기반 의미**
* MySQL 서버가 클라이언트 요청을 처리할 떄 요청마다 별도의 프로세스를 만드는 것이 아니다! 스레드를 만든다는 의미이다.
  * 예) 클라이언트 A가 접속 -> MySQL 서버는 스레드 A 생성 <br>클라이언트 B가 접속 -> MySQL 서버는 스레드 B 생성 
  <br>-> 모두 같은 메모리 공간에서 동작함