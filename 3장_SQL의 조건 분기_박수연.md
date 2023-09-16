# sql의 조건 분기

## union을 사용한 쓸데없이 긴 표현
조건분기는 복수의 조건에 일치하는 하나의 결과 집합을 얻고 싶을때 사용

UNION을 사용하면 눈으로 보기엔 하나의 SQL문을 사용하는것 처럼 보인다
-> 내부적으로 여러개 사용해 성능 낮아진다.

이번 장에서는 UNION, CASE에 대해 설명

EX) year이 2001년 이후는 tax포함된 가격 2001년 이전은 tax제외한 가격을 나타내고 싶을때

#### UNION사용한 쿼리

SELECT item_name, year, price_tax_ex AS price
	FROM Items
 WHERE year <= 2001
UNION ALL
SELECT item_name, year, price_tax_ex AS price
	FROM Items
 WHERE year >= 2002;
 
-> 실행시키면 oracle과 postgresql에서 모두 테이블에 두 번 접근한다.

####  개선된 쿼리

 SELECT item_name, year,
	CASE WHEN year <= 2001 THEN price_tax_ex
    	 WHEN year >= 2002 THEN price_tax_in END AS price
 FROM Items;

 테이블 접근 횟수가 한번

 UNION은 SELECT구문을 기준으로 분기,
 CASE는 식을 바탕으로하는 사고이다.
####  구문보다는 식으로 만드는 사고방식 추천

### UNION이 필요한 경우
1. 다른 테이블의 결과를 합치는 경우
-> CASE를 사용해서 구현이 가능하지만 FROM구에서 테이블을 결합하면 성능이 떨어지기(필요없는 결합이 발생) 때문에 UNION을 사용한다.
2. 인덱스와 관련된 경우
-> 테이블 풀 스캔을 하는 경우 UNION방법이 더 좋다.



SELECT key, name,
	date_1, flg_1,
    date_2, flg_2,
    date_3, flg_3
  FROM ThreeElements
 WHERE date_1 = '2013-11-01'
  AND flg_1 = 'T'
UNION
SELECT key, name,
	date_1, flg_1,
    date_2, flg_2,
    date_3, flg_3
  FROM ThreeElements
 WHERE date_2 = '2013-11-01'
  AND flg_2 = 'T'
UNION
SELECT key, name,
	date_1, flg_1,
    date_2, flg_2,
    date_3, flg_3
  FROM ThreeElements
 WHERE date_3 = '2013-11-01'
  AND flg_3 = 'T';

인덱스 생성

  CREATE INDEX IDX_1 ON ThreeElements (date_1, flg_1);
CREATE INDEX IDX_2 ON ThreeElements (date_2, flg_2);
CREATE INDEX IDX_3 ON ThreeElements (date_3, flg_3);


OR, IN, CASE를 활용했을 경우 1회의 테이블 풀 스캔이 일어나고 인덱스를 활용했을 경우 3회의 인덱스 스캔이 일어난다.


테이블의 크기와 검색 조건에 따른 선택 비율에 따라 답이 달라진다.