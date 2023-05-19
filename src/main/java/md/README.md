<br>

___
## 서브 쿼리

### 서브 쿼리 지원 함수

- [NOT] EXISTS : 서브쿼리에 결과가 존재하면 참
- ALL : 모두 만족하면 참
- ANY, SOME : 하나라도 만족하면 참
- [NOT] IN : 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

```sql
-- 나이가 평균보다 많은 회원
select m from Member m 
         where m.age > (select avg(m2.age) from Member m2);

-- 한 건이라도 주문한 고객
select m from Member m 
         where (select count(o) from Order o where m = o.member) > 0;

-- 팀A 소속인 회원
select m from Member m 
         where exists (select t from m.team t where t.name = '팀A');
         
-- 전체 상품 각각의 재고보다 주문량이 많은 주문들
select o from Order o 
where o.orderAmount > ALL (select p.stockAmount from Product p); 

-- 어떤 팀이든 팀에 소속된 회원
select m from Member m 
where m.team = ANY (select t from Team t);
```

### JPA 서브 쿼리 한계
- JPA 표준 스펙 에선 WHERE, HAVING 절에서만 서브 쿼리 사용 가능
- 하이버네이트에서는 SELECT 절도 가능
- FROM절의 서브 쿼리는 하이버네이트6 부터 가능(JOIN이나 어플리케이션에서 처리)

<br>

___
## JPQL 타입 표현과 기타식

### JPQL 타입 표현
- 문자 : ‘HELLO’, ‘She’’s’
- 숫자 : 10L(Long), 10D(Double), 10F(Float)
- Boolean : TRUE, FALSE
- ENUM : jpabook.MemberType.Admin (패키지명 포함)
- 엔티티 타입 : TYPE(m) = Member (상속 관계에서 사용)

```java
//            em.createQuery("select m.username, 'HELLO',TRUE from Member m" +
//                    " where m.type = jpql.MemberType.USER").getResultList();
// Enum 에 패키지 명을 포함시키기 지저분하니 파라미터 바인딩으로 해결.
em.createQuery("select m.username, 'HELLO',TRUE from Member m" +
        " where m.type = :userType")
        .setParameter("userType", MemberType.USER).getResultList();

// 엔티티 타입
em.createQuery("select i from Item i where type(i) = Book", Item.class).getResultList();


```

### JPQL 기타
- SQL과 문법이 같은 식
- EXISTS, IN
- AND, OR, NOT
- =, >, >=, <, <=, <>
- BETWEEN, LIKE, IS NULL

<br>

___
## 조건식(CASE 등등)

### 조건식 - CASE 식
```sql
-- 기본 case 식
select
    case when m.age <= 10 then '학생요금'
         when m.age >= 60 then '경로요금'
         else '일반요금'
        end
from Member m;

-- 단순 case 식
select
 case t.name 
 when '팀A' then '인센티브110%'
 when '팀B' then '인센티브120%'
 else '인센티브105%'
 end
from Team t;
```

- COALESCE: 하나씩 조회해서 null이 아니면 반환
- NULLIF: 두 값이 같으면 null 반환, 다르면 첫번째 값 반환

```sql
-- 하나씩 조회해서 null이 아니면 반환
select coalesce(m.username,'이름 없는 회원') from Member m;

-- 두 값이 같으면 null 반환, 다르면 첫번째 값 반환
select NULLIF(m.username, '관리자') from Member m;
```

<br>

___
## JPQL 함수

### JPQL 기본 함수
- CONCAT : concat('a','b') || 도 사용가능
- SUBSTRING
- TRIM : 공백제거
- LOWER, UPPER : 대소무낮
- LENGTH : 문자의 길이
- LOCATE : ex) locate('de', 'abcdef') => 4
- ABS, SQRT, MOD
- SIZE, INDEX(JPA 용도)

### 사용자 정의 함수 호출
- 하이버네이트는 사용전 방언에 추가해야 한다.
- 사용하는 DB 방언을 상속받고, 사용자 정의 함수를 등록한다

```java
public class MyH2Dialect extends H2Dialect {

    public MyH2Dialect() {
        registerFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));
    }

}
```
```xml
<!--persistence.xml-->
<property name="hibernate.dialect" value="dialect.MyH2Dialect"/>
```

```java
String query = "select function('group_concat', m.username) from Member m";
//group_concat(m.username) 으로 사용가능

List<String> resultList = em.createQuery(query, String.class).getResultList();

for (String s : resultList) {
    System.out.println("s = " + s);
}
```



