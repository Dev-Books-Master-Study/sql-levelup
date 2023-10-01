## 23강 레코드에 순번 붙이기

### 23.1. PK가 한 개의 필드일 경우

- 윈도우 함수를 사용
- `ROW_NUMBER OVER(ORDER BY 정렬 기준)`

```sql
// 학생 ID를 기준으로 오름차순
SELECT student_id,
		ROW_NUMBER() OVER(ORDER BY student_id) AS seq
	FROM Weights;
```

- `ROW_NUMBER` 사용이 불가한 경우 상관 서브쿼리 사용
- `ROW_NUMBER` 성능이 더 우수

### 23.2. PK가 여러 개의 필드로 구성된 경우

- 윈도우 함수를 사용(`ROW_NUMBER OVER(ORDER BY 정렬 기준)`)

```sql
// 학생 ID를 기준으로 오름차순
SELECT student_id,
		ROW_NUMBER() OVER(ORDER BY class, student_id) AS seq
	FROM Weights;
```

- 상관 서브쿼리도 가능
    - 다중 필드 비교를 사용
    - 다중 필드를 하나의 값으로 연결하고 한꺼번에 비교

### 23.3. 그룹마다 순번을 붙이는 경우

- 윈도우 함수를 사용(`PARTITION BY 필드` 적용)

```sql
// 학생 ID를 기준으로 오름차순
SELECT student_id,
		ROW_NUMBER() OVER(PARTITION BY class ORDER BY class, student_id) AS seq
	FROM Weights;
```

### 23.4. 순번과 갱신

- 윈도우 함수를 사용
- 서브쿼리를 함께 사용

## 24강 레코드에 순번 붙이기 응용

### 24.1. 중앙값 구하기

- 데이터 수가 홀수 → 중앙값
- 데이터 수가 짝수 → 중앙에 있는 두 값의 평균
- 집합 지향적 방법
    
    ```sql
    SELECT AVG(weight)
    	FROM (SELECT W1.weight
    			FROM Weights W1, Weights W2
    		GROUP BY W1.weight
    		HAVING	SUM(CASE WHEN W2.weight >= W1.weight THEN
    			1 ELSE 0 END) >= COUNT(*)/2
    		AND SUM(CASE WHEN W2.weight <= W1.weight THEN
    			1 ELSE 0 END) >= COUNT(*)/2) TMP;
    ```
    
    - 상위 집합과 하위 집합으로 분할 후 공통 부분 검색
    - 코드가 복잡하고 성능이 떨어짐
- 절차 지향적 방법 1 : **양쪽 끝에서 하나씩 조회**
    
    ```sql
    SELECT AVG(weight) AS median
    	FROM (SELECT weight,
    		ROW_NUMBER() OVER (ORDER BY weight ASC, student_id ASC) AS hi,
    		ROW_NUMBER() OVER (ORDER BY weight DESC, student_id DESC) AS lo
    		FROM Weights) TMP
    	WEHRE hi IN(lo, lo+1, lo+2);
    ```
    
    - ROW_NUMBER 함수를 사용해서 순번을 붙임 → 자연수의 연속성과 유일성을 보장하기 위해
    - ORDER BY에 PK 포함 → NULL 방지
- 절차 지향적 방법 1 : 2 빼기 1은 1
    - 절차 지향적 방법 1의 성능을 개선
    - SQL 표준으로 중앙값을 구하는 방법 중 가장 빠름
    
    ```sql
    SELECT AVG(weight)
    	FROM (SELECT weight,
    			2 * ROW_NUMBER() OVER(ORDER BY weight)
    			- COUNT(*) OVER() AS diff
    		FROM Weights) TMP
    	WHERE diff BETWEEN 0 AND 2;
    ```
    

### 24.1. 순번을 사용한 테이블 분할

- 빈 좌석 찾기
- 집합 지향적 방법 : 집합의 경계선
    
    ```sql
    SELECT (N1.num + 1) AS gap_start,
    	   '~',
    	   (MIN(N2.min - 1) AS gap_end
    	FROM Number N1 INNER JOIN Numbers N2
    	ON N2.num > N1.num
    		GROUP BY N1.num
    	HAVING (N1.num + 1) < MIN(N2.num);
    ```
    
    - 코드가 간단하고, 집합 집약적 방법이라 좋음
    - 자기 결합을 사용해야함 → Nested Loops 결합이 발생해 쿼리 비용이 높음
- 절차 지향적 방법 : 다음 레코드와 비교
    - 현재 레코드와 다음 레코드의 숫자 차이가 1이 아니면 → 중간이 비어있음
    
    ```sql
    SELECT NUM+1 AS gap_start,
    	   '~',
    	   (num + diff - 1) As gap_end
    	FROM (SELECT num,
    			MAX(num)
    				OVER(ORDER BY num
    					ROWS BETWEEN 1 FOLLOWING
    							AND 1 FOLLOWING) - num
    		FROM numbers) TMP (num, diff)
    	WHERE diff <> 1
    ```
    
    - 테이블 접근 1회, 정렬 1회 → 안정적인 성능

### 24.3. 테이블에 존재하는 시퀀스 구하기

- 빈 자리의 덩어리 구하기
- 집합 지향적 방법 : 집합의 경계선
    
    ```sql
    SELECT MIN(NUM) AS low,
    	   '~',
    	   MAX(num) AS high
    	FROM (SELECT N1.num,
    			COUNT(N2.num) - N1.num
    		FROM Numbers N1 INNER JOIN Numbers N2
    			ON N2.num <= N1.num
    			GROUP BY N1.num) N(num, gp)
    		GROUP BY gp;
    ```
    
    - 자기 결합을 수행
    - 정렬 대신 해시를 사용
- 절차 지향적 방법 : 다음 레코드와 비교
    
    ```sql
    SELECT low, high
    	FROM(SELECT low,
    			CASE WHEN high IS NULL
    				THEN MIN(high)
    					OVER (ORDER BY seq
    						ROW BETWEEN CURRENT ROW
    							AND UNBOUNDED FOLLOWING)
    				ELSE high END AS high
    		FROM (SELECT CASE WHEN COALESCE(prev_diff, 0) <> 1
    						THEN num ELSE NULL END AS low,
    					CASE WHEN COALESCE(next_diff, 0) <> 1
    						THEN num ELSE NULL END As high,
    					seq
    			FROM (SELECT num,
    						MAX(num)
    						OVER(ORDER BY num
    								ROWS BETWEEN 1 FOLLOWING
    									AND 1 FOLLOWING) - num AS next_diff,
    						num - MAX(num)
    							OVER(ORDER BY num
    									ROWS BETWEEN 1 PRECEDING
    										AND 1 PRECEDING) AS prev_diff,
    						ROW_NUMBER() OVER (ORDER BY num) AS seq
    					FROM Numbers) TMP1 ) TMP2 ) TMP3
    WHERE low IS NOT NULL;
    ```
    

## 25강 시퀀스 객체, IDENTITY 필드, 채번 테이블

- 표준 SQL에는 순번을 다루는 기능으로 시퀀셜 객체나 IDENTIFY 필드가 존재
- 하지만 최대한 사용하지 않는 것을 권장, 사용한다면 시퀀스 객체를 사용

### 25.1. 시퀀스 객체

- 스키마 내부에 존재하는 객체 중 하나
- 시퀀스 객체의 문제점
    - 표준화가 늦음 → 구현에 따라 구문이 달라 이식성이 없고 사용할 수 없는 구현도 존재
    - 실제 엔티티 속성이 아님
    - 성능 문제 발생
- 시퀀스 객체로 발생하는 성능 문제
    - 유일성 : 1, `2, 2`, 3, …
    - 연속성 : 1, `2, 4,` 5, `6, 8`, …
    - 순서성 : 1, 2, `5, 4`, 6, …
    - 위 문제를 해결하기 위해 락 메커니즘이 필요 → 락 충돌로 성능 저하 문제가 발생할 수 있음
- 시퀀스 객체로 발생하는 성능 문제 대처
    - CACHE, NOORDER
    - CACHE
        - 읽어들일 변수를 메모리에 설정
        - 시스템 장애시 정상동작을 담보할 수 없음
    - NOORDER
        - 순서성을 담보하지 않음 → 오버 헤드 감소
- 순번을 키로 사용할 때의 성능 문제
    - 핫 스팟과 관련된 문제
    - DBMS는 비슷한 데이터를 연속으로 INSERT하면 물리적으로 같은 영역에 저장됨
    - 특정 물리적 블록에만 I/O 부하가 커짐 → 성능 악화
- 순번을 키로 사용할 때의 성능 문제 대처
    - Oracle의 열 키 인덱스
        - 연속된 값을 도입하는 경우 DBMS 내부에 변화를 주어 제대로 분산할 수 있는 구조를 사용
        - SELECT 성능이 나빠질 수 있음
    - 인덱스에 복잡한 필드를 추가해서 데이터의 분산도를 높이기
        - INSERT 구문은 빨라짐
        - 범위 검색 등에서 I/O 양이 늘어나 SELECT 구문 성능이 나빠질 수 있음
        - 논리적으로 좋은 설계가 아님

### 25.2. **IDENTIFY 필드**

- 자동 순번 필드
- 테이블의 필드로 정의하고, INSERT 발생할때마다 자동을 순번을 붙여주는 기능
- 시퀀스 객체에 비해 단점이 많음
    - 시퀀스 객체는 여러 테이블에서 사용가능하지만, IDENTIFY 필드는 특정 테이블에 한정됨
    - CACHE, NOORDER를 지정할 수도 없음
    - 이점이 거의 없음
    

### 25.3. 채번 테이블

- 구시대 유물
- 병목 현상이 발생하지 않기를 기도하기