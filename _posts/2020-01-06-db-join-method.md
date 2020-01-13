---
layout : post
title : "Database: Join Method"
date : 2020-01-06
category :
  - DB
elements:
  - join
  - database
  - hash
  - nested loop
  - sort merge
comments : true

---

# SQL 튜닝[조인 방법]

Join 방법의 분류

# 조인 방법

---

쿼리에 조인이 포함되어 있을 때 옵티마이저가 판단하는 세 가지 조인 연산 방법(Join Method)을 통하여 조인 작업을 수행하게 된다.
Nested Loop Join, Sort Merge Join, Hash Join, Single Loop Join 의 방법으로 두 릴레이션으로부터 관련된 튜플들을 결합하여 하나의 튜플로 만드는 데이터 연결 방법을 JOIN 연산이라 한다.

## Nested Loop Join (중첩 루프 조인)

---

### 개념

조인되는 두개의 테이블 중 하나의 테이블을 기준으로 기준이 아닌 다른 테이블의 데이터를 읽는다.

순차적으로 Inner Table의 Row를 결합하여 원하는 결과를 조합하는 방식이다.

JAVA의 for문 안에 for문이 도는 형태이다.

    for(i=0; i<100; i++){  -- outer loop
    	for(j=0; j<100; j++){  -- inner loop
    		// Anything
    	}
    }

### Nested Loop 특징

- 첫 번째 Row를 받는 시간이 빠르지만, 전체 결과를 받는데까지는 시간이 걸림
- Nested Loop Join은 메모리가 필요없는 조인 방법이며 추가적인 메모리 비용이 들지 않는다.
- 좁은 범위에 유리한 성능
- 순차적 처리, Random Access 위주
- Driven Table에는 조인을 위한 인덱스 생성 필요
- 실행속도  = Driving Table Size * Driven Table Access
- Nested Loop Join 방식은 Inner Table의 Index 효율이 좋아야 한다. Driving Table을 조회할때는 Full Table Scan이나 Index Scan을 사용한다.
- 따라서 Driving Table 의 사이즈가 성능에 영향을 많이 미친다.

### Nested Loop 사용 시 주의사항

- Random Access 위주이기 때문에 결과 집합이 많으면 속도가 느려진다.
- Join Index가 없거나, 조인 집합을 구성하는 검색조건이 조인 범위를 줄여주지 못할 경우 비효율적이다.

## Sort Merge Join (정렬 병합 조인)

---

두 테이블을 각각 Sort 한 다음 조인조건에 맞는 건을 찾아 merge 하는 방식의 조인으로 Driving Table이 존재하지 않고, 각각의 Table이 모두 독립적으로 동등한 레벨에 있다.

### 개념

조인의 대상범위가 넓을 경우 발생하는 Random Access를 줄이기 위해 사용하며

조인을 위한 연결고리에 마땅한 인덱스가 존재하지 않을 경우 사용한다.

양쪽 Table을 각각 Access 하여 정렬한 결과를 차례로 Scan하고 열결고리 조건으로 Merge시킨다.

### Sort Merge Join 특징

- 랜덤 액세스를 하지 않고 스캔을 하면서 수행한다.
- 선행집합 개념이 없다.
- Sort Area Size에 따라 효율에 큰 차이가 발생한다.
- 첫 번째 로우를 받는 시간은 느리지만, 전체 로우가 반환되는 시간은 빠르다.  (각각의 테이블의 데이터를 우선적으로 Sort 해야하기 때문에 Sort 작업이 끝나기 전까지 첫 번째 로우가 반환되지 않는다.)
- Nested Loop Join보다 많은 양의 데이터를 처리할 때 유리하고, 메모리를 사용해서 정렬 작업을 수행한다면 넓은 범위의 값을 검색하는데 유리하다.
- Sort 하는 작업이 전체 성능에 영향을 많이 주기 때문에 SELECT 리스트에서 불필요한 컬럼은 제거하여 Sort 작업의 부하를 적게 하여야 한다.
- 조인조건에 사용되는 연산자가 비동등 연산자 일 경우 Sort Merge Join을 사용하게 된다.

## Hash Join (해쉬 조인)

---

두 데이블 중 WHERE 조건에 의해 필터링 되어 로우수가 적은 Table을 대상으로 해쉬 테이블을 만든 뒤 조인 조건에 따라 남은 테이블의 데이터를 검색하는 방법

### 개념

조인대상이 되는 두 집합 중에서 작은 집합(Build Input)을 읽어 Hash Area에 해시 테이블을 생성한다. 그 후 큰 집합(Probe Input)을 읽어 해시 테이블을 탐색하면서 조인한다.

**작은집합**(**Build Input)을 읽어 해시맵 생성**

- 해시테이블을 생성할 때 해시함수를 사용하고, 해시함수에서 리턴받은 버킷주소로 찾아가 해시체인에 엔트리를 연결

**큰 집합(Probe Input)을 스캔**

- 해시테이블을 탐색할 때 해시함수를 사용하고, 해시함수에서 리턴받은 버킷주소로 찾아가 해시체인을 스캔하면서 데이터를 서칭

### Hash Join 특징

- Nested Loop Join은 Random Access로 인한 부하가 존재하지만 Hash Join은 Random 엑세스 부하가 없다. 또한 Sort Merge Join처럼 각 테이블의 Sort에 대한 부담 또한 존재하지 않는다.
- 해시테이블을 생성하는 비용이 들어가기 때문에 Build Input의 크기가 작아야 효율이 좋다.
- *해시키 값으로 사용되는 컬럼에 중복값이 없을 경우 효과적이다.*
- Hash Join은 조인 Column의 인덱스가 존재하지 않을 때도 사용할 수 있다.
- Hash Join 작업을 하려면 해쉬 테이블을 메모리에 생성해야하고 메모리의 적재 용량을 초과하면 임시영역(디스크)에 해쉬 테이블을 저장한다.
