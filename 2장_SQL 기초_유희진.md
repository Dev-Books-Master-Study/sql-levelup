# SELECT 구문

- 데이터베이스를 이용할 때 핵심이 되는 처리가 바로 검색이다.
- 다른 말로 질의(query), 추출(retrieve)이라고 부른다.
- 검색을 위해 사용하는 구문을 `SELECT` 구문이라고 부른다.

## SELECT 구와 FROM 구

```sql
SELECT id, name FROM tb_user
```

- 첫번째 부분은 `SELECT` 뒤에 나열되어 있는 부분으로 `SELECT` 구라고 부른다.
- 두번쨰 부분은 `FROM table_name` 형식으로 FROM구라고 부른다.
    - 반드시 입력해야할 부분은 아니지만 테이블에서 데이터를 가져올 경우 반드시 입력해야한다.
- `FROM`구를 입력하지 않아도 되는 경우는 다음과 같다.
    1. 상수를 선택하는 경우
        
        ```sql
        SELECT 1
        ```
        
    2. 계산 결과를 보고 싶은 경우
        
        ```sql
        SELECT DATEDIFF(DAY, '2017-02-13', '2017-03-15') AS RESULT # 30
        ```
        
- `SELECT`구문에서는 **데이터를 어떤 방법으로 선택할지 쓰여있지 않다. - 실행 계획**
    - 이 방법은 DBMS에게 맡긴다.
    - [https://velog.io/@sweet_sumin/쿼리-실행-계획](https://velog.io/@sweet_sumin/%EC%BF%BC%EB%A6%AC-%EC%8B%A4%ED%96%89-%EA%B3%84%ED%9A%8D)

## WHERE 구

```sql
SELECT name, address
FROM address
WHERE address = '인천시';   # 주소가 인천시인 사람들의 정보만 가져온다.
```

- 항상 모든 레코드가 필요하지 않고, 특정 조건에 맞는 일부 레코드만 필요한 경우가 많다.
- 이러한 경우를 위해 `WHERE`구를 사용하여 추가적인 조건을 지정한다.

### WHERE 구 조건 지정

| 연산자 | 의미 |
| --- | --- |
| = | ~와 같다 |
| <> | ~와 같지 않다 |
| ≥ | ~이상 |
| > | ~보다 큼 |
| ≤ | ~ 이하 |
| < | ~ 보다 작음 |

### WHERE 구는 거대한 벤다이어그램

- 실제 사용하는 조건은 상당히 복합적일 때가 많다.
- 예를 들어, `‘주소가 서울시에 있다’ 그리고 ‘나이가 30세 이상이다’`라는 복합 조건을 생각해보자.
- 이 때 두 개의 조건을 연결하는 ‘그리고’를 표현해야하는데, 이는 `‘AND’`를 사용하면 된다.
    
    ```sql
    SELECT name, address
    FROM address
    WHERE address = '서울시';
    	AND age = 30;
    ```
    
- `AND`에 상응하는 `OR`도 존재한다.
    
    ```sql
    SELECT name, address
    FROM address
    WHERE address = '서울시';
    	OR age = 30;
    ```
    

### IN 조건으로 OR 조건을 간단하게 작성

- 상황에 따라서는 `OR` 조건을 굉장히 많이 지정해야할 때가 있을 수 있다.
    
    ```sql
    SELECT name, address
    FROM address
    WHERE address = '서울시'
    	OR address = '부산시'
    	OR address = '인천시';
    ```
    
- SQL은 이러한 조건을 간단하게 작성할 수 있도록 `IN` 연산자를 제공한다.
    
    ```sql
    SELECT name, address
    FROM address
    WHERE address IN ('서울시', '부산시', '인천시');
    ```
    

### NULL - 아무것도 아니라는 것은 무엇일까?

- `WHERE` 구로 조건을 지정할 때 초보자가 처음 곤란해하는 부분이 `NULL`을 검색할 때이다.
- 보통 `NULL`인 사람들만 선택하는 조건을 만들고 싶다면 다음과 같이 작성한다.
    
    ```sql
    SELECT name, address
    FROM address
    WHERE phone_nbr = NULL;
    ```
    
    - 하지만 이는 정상적인 접근이 아니다.
    - 오류가 발생하지는 않지만 결과로 아무것도 나오지 않는다.
- `NULL` 레코드를 선택할 땐 `IS NULL`이라는 특별한 키워드를 사용해야한다.
    
    ```sql
    SELECT name, address
    FROM address
    WHERE phone_nbr IS NULL;
    ```
    
    - 반대로 `NULL`이 아닌 레코드를 선택하고 싶은 경우 `IS NOT NULL`이라는 키워드를 사용한다.
    - [https://velog.io/@minnim1010/SQL-NULL-이-아닌-IS-NULL인-이유-9irf8u7o](https://velog.io/@minnim1010/SQL-NULL-%EC%9D%B4-%EC%95%84%EB%8B%8C-IS-NULL%EC%9D%B8-%EC%9D%B4%EC%9C%A0-9irf8u7o)

## SELECT 구문은 절차 지향형 언어의 함수

- `SELECT` 구문의 기능은 절차 지향형 언어에서의 ‘함수’와 같다는 것을 알 수 있다.
- 함수라는 것은 모두 알고 있는 것처럼 입력을 받고 관련된 처리를 수행한 뒤에 리턴하는 것이다.
- `SELECT` 구문도 테이블이라는 입력을 `FROM` 구로 받아 특정 출력을 리턴한다는 점에서 같은 방식으로 작동한다.
    - 이때, 입력이 되는 테이블에는 변경이 일어나지 않기 때문에 `SELECT` 구문은 **일종의 읽기 전용 함수**라고 말할 수 있다.
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/dc805bdb-ff59-4dd2-9416-9b8fa9c8b78f/b43344f1-c049-491e-864a-53c90f917e69/Untitled.jpeg)
        
- 일반적인 절차 지향형 언어의 함수는 매개변수와 리턴값의 자료형이 정수 자료형 또는 문자열 자료형처럼 결정되어 있다.
    - `SELECT`도 자료형이 결정되어 있는데, 출력 자료형은 바로 테이블이다.
    - 2차원 표 이외에는 어떠한 자료형도 존재하지 않는다.
    - 이러한 성질 때문에 **관계가 닫혀있다는 의미로 폐쇄성(closure property)**이라고 부른다.

## GROUP BY 구

- 합계, 평균 등의 집계 연산이 가능해진다.
- ‘테이블을 홀 케이크처럼 다룬다’라고 할 수 있다.
    - 칼이라는 도구를 사용해 케이크를 자르고 많은 사람이 나눠먹는 것이 일반적인데, `GROUP BY`구는 이 케이크를 자르는 칼과 같은 역할을 한다.
        
        ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/dc805bdb-ff59-4dd2-9416-9b8fa9c8b78f/cff30d7c-362c-4114-8aee-8f8f3cec40c6/Untitled.jpeg)
        

### 그룹을 나누었을 때의 장점

- 잘라진 케이크 조각을 그룹이라고 부른다.
- 그리고 이러한 그룹은 다양한 숫자 관련 함수를 사용한 집계가 가능하다.
- 대표적인 집계 함수는 다음과 같다.
    
    
    | 함수 이름 | 설명 |
    | --- | --- |
    | COUNT | 레코드 수를 계산 |
    | SUM | 숫자를 더함 |
    | AVG | 숫자의 평균을 구한다. |
    | MAX | 최댓값을 구한다. |
    | MIN | 최솟값을 구한다. |
- 예를 들어, 남자 그룹과 여자 그룹 각각 몇명이 있는지 구하고 싶다면 다음과 같이 사용한다.
    
    ```sql
    SELECT sex, COUNT(*)
    FROM address
    GROUP BY sex;
    ```
    
    - [추가] `COUNT(col_name)` - NULL값 제외, `COUNT(*)` - NULL값 포함
- `GROUP BY`를 사용하면서 나누지 않는 경우는 다음과 같이 사용한다.
    
    ```sql
    SELECT COUNT(*)
    FROM address
    GROUP BY ();
    ```
    
    - 보통 이런 경우는 `GROUP BY`를 생략한다.

## HAVING

```sql
SELECT address, COUNT(*)
FROM address
GROUP BY address
HAVING COUNT(*) = 1;   # 해당 주소에 살고 있는 사람이 한 명인 주소만 선택한다.
```

- `GROUP BY`에 조건을 걸 때 사용한다.
- `HAVING` 구를 사용하면 선택된 결과 집합에 또 다시 조건을 지정할 수 있다.
- `**WHERE` 구가 레코드에 조건을 지정한다면, `HAVING` 구는 집합에 조건을 지정하는 기능**이다.
    - Having은 그룹화 또는 집계가 발생한 후 레코드를 필터링하는데 사용된다.
    - Where은 그룹화 또는 집계가 발생하기 전에 레코드를 필터링하는데 사용된다.
    - [https://velog.io/@ljs7463/SQL-having-과-where-차이](https://velog.io/@ljs7463/SQL-having-%EA%B3%BC-where-%EC%B0%A8%EC%9D%B4)

## ORDER BY 구

```sql
SELECT address, age, name
FROM address
ORDER BY name, age DESC
```

- 위 쿼리는 name으로 오름차순 정렬을 하고, 동명이인이 있을 경우엔 나이순으로 내림차순 정렬을 한다.
- `SELECT` 출력 결과의 순서는 정해진 규칙은 없다.
- 모든 DBMS에서 `SELECT` 구문의 결과 순서를 보장하려면 명시적으로 순서를 지정해줘야한다.
    - 이 때 `ORDER BY`를 사용한다.
- 오름차순은 `ASC(default)`, 내림차순은 `DESC` 옵션을 붙여주면 된다.

## 뷰와 서브쿼리

- DB를 사용하다보면 `SELECT` 구문 중에서 자주 사용하는 것과 거의 사용하지 않는 것이 나온다.
- 이때 **자주 사용하는 구문을 DB안에 저장할 수 있는데, 이 기능을 뷰(View)**라고 한다.
    - View는 데이터베이스안에 저장한다는 점은 테이블과 같지만, **내부적으로 데이터를 보유하지 않는다.** (일종의 가상 테이블)
    - View는 `SELECT` 구문을 저장한 것뿐이다.
    - 그래서 **디스크 저장 공간 할당이 이루어지지 않는다.**
    - https://reeme.tistory.com/54

### 뷰 만들기

```sql
CREATE VIEW [뷰 이름] (필드이름1, 필드이름2, ...) AS [SELECT구문]

CREATE VIEW count_address (address, cnt)
AS
SELECT address, COUNT(*)
FROM address
GROUP BY address;
```

- 이렇게 만들어진 뷰는 일반적인 테이블처럼 `SELECT` 구문에서 사용할 수 있다.
- 뷰라는 것은 한 마니도 테이블의 모습을 한 `SELECT` 구문이라고 할 수 있다.

### 익명 뷰

- 뷰는 사용 방법이 테이블과 같지만 내부에는 데이터를 보유하지 않는다는 점이 테이블과 다르다.
- 따라서 뷰에서 데이터를 선택하는 `SELECT` 구문은 실제로는 내부적으로 추가적인 `SELECT` 구문을 실행하는 중첩 구조가 된다.

```sql
SELECT address, cnt
FROM count_address;

SELECT address, cnt
FROM (
	SELECT address, COUNT(*) AS cnt
	FROM address
	GROUP BY address
) as count_address;
```

- 이렇게 FROM구에 직접 지정하는 SELECT 구문을 서브쿼리라고 한다.

### 서브쿼리를 사용한 편리한 조건 지정

```sql
SELECT name
FROM address
WHERE name IN (SELECT name FROM address2);
```

- WHERE 구 조건에 서브쿼리를 사용하는 방법이 있다.
- 위 문장은 **두 테이블을 사용해 address 테이블에서 address2 테이블에 있는 사람을 선택하는 문장**이다.
    - 이러한 처리를 매칭(matching)이라고 부른다.
- IN은 상수뿐만 아니라 서브쿼리도 매개변수로 받을 수 있다.
- SQL은 서브쿼리부터 순서대로 실행하며, SELECT 구문을 받은 DBMS는 서브쿼리를 상수로 전개해서 아래와 같이 바꾼다.
    
    ```sql
    SELECT name
    FROM address
    WHERE name IN ('인성', '민', '준서', '지연', ...)
    ```
    

---

# 조건 분기, 집합 연산, 윈도우 함수, 갱신

## SQL과 조건 분기

- 일반적인 절차지향형 프로그래밍 언어에는 조건 분기를 사용하기 위한 수단으로 `if` 조건문과 `switch` 조건문이 있다.
- SQL에서도 조건을 분기하는 방법이 있지만 사용법이 다르다.
    - 이는 SQL은 코드를 절차적으로 기술하는 것이 아니기 때문에 조건 분기를 ‘문장’ 단위로 하지 않기 때문이다.
    - SQL은 조건 분기를 ‘식’으로 하며, 이러한 식의 분기를 실현하는 기능이 바로 `CASE` 식이다.

### CASE 식의 구문

- `CASE` 식의 구문에는 단순 `CASE` 식과 검색 `CASE` 식이라는 두 종류가 있다.
- 다만 검색 `CASE` 식은 단순 `CASE` 식의 기능을 모두 포함하고 있으므로 검색 `CASE` 식만 기억해도 된다.
- 검색 CASE 식 구문
    
    ```sql
    CASE WHEN [평가식] THEN [식]
    		 WHEN [평가식] THEN [식]
    		 WHEN [평가식] THEN [식]
    		 ELSE [식]
    END
    
    # 예시
    SELECT email,
    CASE
    	WHEN age >= 20 AND age < 30 THEN '이십 대'
      WHEN age BETWEEN 30 AND 39 THEN '삼십 대'
      ELSE "기타"
    END AS "나이 대"   # AS로 새로운 필드명을 지정한다.
    FROM main_database.member;
    ```
    
- 단순 CASE 식 구문
    
    ```sql
    CASE 컬럼 이름
    	WHEN 값 THEN 값
    	WHEN 값 THEN 값
      WHEN 값 THEN 값
      ELSE 값
    END
    
    # 예시
    SELECT email,
    CASE age 
    	WHEN 22 THEN '스물 두 살'
      WHEN 32 THEN '서른 두 살'
      ELSE age
    END AS "나이"
    FROM main_database.member;
    ```
    

[https://velog.io/@dong5854/SQL-CASE-함수](https://velog.io/@dong5854/SQL-CASE-%ED%95%A8%EC%88%98)

### CASE 식의 작동

- `CASE` 식의 작동은 `switch` 조건문과 비슷하다.
- `switch` 조건 분기와 SQL 조건 분기 사이의 가장 큰 차이점은 바로 리턴값이다.
    - 조건 분기는 문장을 실행하고 (별도의 리턴을 선언하지 않으면) 딱히 리턴하지는 않지만, SQL 조건 분기는 (리턴을 선언하지 않아도) 특정값을 리턴한다.
    - https://wiki.webnori.com/display/webfr/SQL+PART-B
- `CASE` 식의 장점은 바로 ‘식’이라는 것이다.
    - 따라서 식을 적을 수 있는 곳이라면 어디든지 적을 수 있다.
    - `SELECT`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY` 등 어느 곳에나 적을 수 있기 때문에 다양한 기법으로 활용할 수 있다.

## SQL의 집합 연산

- `WHERE` 구에서는 합집합을 `OR`, 교집합을 `AND`가 담당했다.
- 그러나 집합 연산에서는 연산자가 다르다.

### UNION(합집합)

```sql
SELECT * FROM address
UNION
SELECT * FROM address2
```

- 두 개의 테이블을 하나의 테이블로 합친 결과가 나온다.
- 그러나 `UNION` 연산은 중복되는 데이터를 제거하고 나온다.
    - `INTERSECT`, `EXCEPT`에서도 마찬가지이다.
- 만약 중복을 제거하고 싶지 않다면 `UNION ALL`을 사용한다.

### INTERSECT(교집합)

```sql
SELECT * FROM address
INTERSECT
SELECT * FROM address2
```

- 양쪽 테이블에 공통으로 존재하는 레코드를 출력한다.
- 중복되는 요소는 제거한다.

### EXCEPT(차집합)

```sql
SELECT * FROM address
INTERSECT
SELECT * FROM address2
```

- 이를 수식으로 나타내면 `address - address2`가 된다.
- address 데이터에서 address와 address2의 교집합이 제거된다.
- `EXCEPT` 연산은 다른 연산과 달리 순서가 다르면 결과도 다르다는 점에 유의해야한다.

## 윈도우 함수

- 집약 기능이 없는 `GROUP BY` 구이다.
    - 이전에 보았던 `GROUP BY` 구는 자르기와 집약이라는 두 개의 기능으로 구분되는데, 윈도우 함수는 여기서 자르기 기능만 존재한다.

```sql
SELECT address, COUNT(*)
FROM address
GROUP BY address;
```

- 위 SQL은 address 필드로 테이블을 자르고 이어서 잘린 조각 개수만큼 레코드 수를 더해 결과를 출력한다.
- 윈도우 함수도 테이블을 자르는 것은 같지만, 윈도우 함수는 이를 `PARTITION BY`라는 구로 수행한다.
    - 차이점이 있다면 **자른 후 집약은 하지 않으므로 출력 결과의 레코드 수가 입력되는 테이블의 레코드 수와 같다는 것**이다.
- 윈도우 함수의 기본적인 구문은 집약 함수 뒤에 `OVER` 구를 작성하고, 내부에 자를 키를 지정하는 `PARTITION BY`와 `ORDER BY`를 입력하는 것이다.
    
    ```sql
    SELECT address, COUNT(*) OVER(PARTITION BY address)
    FROM address;
    ```
    
    - `PARTITION BY`와 `ORDER BY`는 둘 중에 하나만 입력해도 되고 둘 다 입력해도 상관없다.
    - 작성 위치는 `SELECT` 구이다.
- GROUP BY와 윈도우 함수 결과 차이 (https://mizykk.tistory.com/121)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/dc805bdb-ff59-4dd2-9416-9b8fa9c8b78f/0ade1da6-ea30-422a-9a13-f0f80ff24ec3/Untitled.png)

![GROUP BY](https://prod-files-secure.s3.us-west-2.amazonaws.com/dc805bdb-ff59-4dd2-9416-9b8fa9c8b78f/5b9edc98-49b1-4a04-82c6-85f8240aba66/Untitled.png)

GROUP BY

![윈도우 함수](https://prod-files-secure.s3.us-west-2.amazonaws.com/dc805bdb-ff59-4dd2-9416-9b8fa9c8b78f/058ae205-eea8-45ac-8998-f0bd546f5b59/Untitled.png)

윈도우 함수

- 윈도우 함수로 사용할 수 있는 함수는 `COUNT` 또는 `SUM` 같은 일반 함수 이외에도 윈도우 함수 전용으로 제공되는 `RANK`, `ROW_NUMBER` 등의 순서 함수도 있다.
    - `RANK` 함수는 이름 그대로 지정된 키로 레코드에 순위를 붙이는 함수이다.
    - 예를 들어 나이가 많은 순서로 순위를 붙인다면 아래와 같이 작성한다.
        
        ```sql
        SELECT name, age, RANK() OVER(ORDER BY age DESC) AS rnk
        FROM address;
        
        -- 실행 결과
        name | age | rnk
        ----------------
        하린 | 55 | 1
        준   | 45 | 2
        기주 | 32 | 3
        민   | 32 | 3
        인성 | 30 | 5   # 4위를 건너뛰고 5위
        ...
        ```
        
        - `RANK` 함수는 숫자가 같으면 같은 순위로 표시하므로 기주와 민이 같은 순위로 매겨진다.
        - 만약 순위를 건너뛰는 작업 없이 순위를 구하고 싶을 땐 `DENSE_RANK` 함수를 사용한다.
    

## 트랜잭션과 갱신

- 기본적으로 SQL의 갱신 작업은 다음과 같다.
    1. 삽입(insert)
    2. 제거(delete)
    3. 갱신(update)

### INSERT

```sql
INSERT INTO [테이블 이름] ([필드1], [필드2], ...) VALUES ([값1], [값2], ..);
```

- 데이터를 등록하는 단위인 레코드를 삽입한다.
- **필드 리스트와 값 리스트는 같은 순서로 대응해야한다.**
- 테이블에 데이터를 삽입하는 구문은 기본적으로 레코드를 하나씩 삽입한다.
    - 따라서 여러 데이터를 삽입하고 싶을 땐 `INSERT`를 여러번 사용해야한다.
    - 최근에는 성능 개선을 위해 여러 레코드를 한 개의 `INSERT` 구문으로 삽입하는 기능을 지원한다. (bulk insert)
        
        ```sql
        INSERT INTO table VALUES (1, "hello"), (2, "world"), (3, "!");
        ```
        

### DELETE

```sql
DELETE FROM [테이블명];
```

- `DELETE`는 `INSERT`와 달리 레코드 단위가 아니라 **한 번에 여러 개의 레코드 단위로 처리하게 된다.**
- 부분적으로 레코드를 제거하고 싶다면 `WHERE`구로 제거 대상을 선별한다.
    
    ```sql
    DELETE FROM address WHERE address = '인천시';
    ```
    
- 아래와 같이 작성하는 경우 오류가 발생한다.
    
    ```sql
    DELETE name FROM address
    ```
    
    - 이는 DELETE 구문의 삭제 대상이 아니라 레코드이기 떄문에 일부 필드만 삭제하는 것이 불가능하기 때문이다.
- DELETE를 통해 모든 데이터를 삭제했다고해도 테이블 자체가 사라지는 것은 아니다.
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/dc805bdb-ff59-4dd2-9416-9b8fa9c8b78f/548173ff-0885-4402-86bf-be529e28a77f/Untitled.png)
    
    | 구분 | 테이블 정의 | 저장공간 | 작업속도 | SQL문 종류 구분 |
    | --- | --- | --- | --- | --- |
    | DELETE | 존재 | 유지 | 느림 | DML |
    | TRUNCATE | 존재 | 반납 | 빠름 | DDL |
    | DROP | 삭제 | 반납 | 빠름 | DDL |
    
    [https://prinha.tistory.com/entry/SQL-DELETE-TRUNCATE-DROP-차이점](https://prinha.tistory.com/entry/SQL-DELETE-TRUNCATE-DROP-%EC%B0%A8%EC%9D%B4%EC%A0%90)
    
    https://goddaehee.tistory.com/55
    

### UPDATE

```sql
UPDATE [테이블명] SET [필드 이름] = [식]
```

- `UPDATE` 구문도 일부 레코드만 갱신하고 싶을 땐 `WHERE`구로 필터링할 수 있다.
- `SET` 구에 여러 개의 필드를 입력하면 한 번에 여러 개의 값을 변경할 수 있다.
    - 따라서 여러 개의 필드를 갱신하고 싶을 때 `UPDATE` 구문을 여러 번 사용할 필요가 없다.