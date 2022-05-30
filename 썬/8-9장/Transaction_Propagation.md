# 트랜잭션 전파 (Transaction Propagation)

트랜잭션 동작 도중 다른 트랜잭션을 호출(실행)하는 상황이에 선택할 수 있는 옵션을 일컫는다.

`@Transactional`의 전파 옵션을 통해 피호출 트랜잭션 입장에서는 호출한 쪽의 트랜잭션을 그대로 사용할 수도, 새롭게 트랜잭션을 생성할 수도 있다.

1. Required
   **디폴트 설정**  
   부모 트랜잭션 내에서 실행하며, 부모 트랜잭션이 없다면 새로운 트랜잭션을 생성한다.

```java
//REQUIRED pseudo-code
if (isExistingTransaction()) {
    if (isValidateExistingTransaction()) {
        validateExisitingAndThrowExceptionIfNotValid();
    }
    return existing;
}
return createNewTransaction();
```

2. MANDATORY
   REQUIRED와 비슷하게 이미 시작된 트랜잭션이 있으면 해당 트랜잭션에 덧붙인다.  
   반면에 활성화된 트랜잭션이 없다면 새로 시작하지 않고 예외를 발생시킨다.  
   (독립적으로 실행되면 안되는 트랜잭션에 사용)
```java
//MANDATORY pseudo-code
if (isExistingTransaction()) {
    if (isValidateExistingTransaction()) {
        validateExisitingAndThrowExceptionIfNotValid();
    }
    return existing;
}
throw IllegalTransactionStateException;
```

3. REQUIRES_NEW
   항상 새로운 트랜잭션을 시작한다. 이미 진행 중인 트랜잭션이 있으면 트랜잭션을 잠시 보류시킨다.
```java
//REQUIRES_NEW pseudo-code
if (isExistingTransaction()) {
    suspend(existing);
    try {
        return createNewTransaction();
    } catch (exception) {
        resumeAfterBeginException();
        throw exception;
    }
}
return createNewTransaction();
```

4. SUPPORTS
   이미 시작된 트랜잭션이 있으면 해당 트랜잭션을 사용하고 그렇지 않으면 트랜잭션 없이 로직을 수행한다.
```java
//SUPPORTS pseudo-code
if (isExistingTransaction()) {
    if (isValidateExistingTransaction()) {
        validateExisitingAndThrowExceptionIfNotValid();
    }
    return existing;
}
return emptyTransaction;
```

5. NOT_SUPPORTED
   트랜잭션을 사용하지 않게 한다.  
   또한, 이미 활성화된 트랜잭션이 존재할 경우, 보류시킨다.

6. NEVER
   트랜잭션을 사용하지 않도록 강제한다.  
   이미 진행 중인 트랜잭션이 있다면 예외를 발생시킨다.
```java
//NEVER pseudo-code
if (isExistingTransaction()) {
    throw IllegalTransactionStateException;
}
return emptyTransaction;
```

7. NESTED
   이미 진행중인 트랜잭션이 있으면 해당 트랜잭션의 세이브 포인트를 기록한다.  
   이후, 해당 트랜잭션 내에 중첩 트랜잭션을 만들어 작업하는데, 만약 그 과정에서 비즈니스 로직에 예외가 발생하면 세이브 포인트로 롤백한다.  
   즉, 하위 트랜잭션은 상위 트랜잭션에 영향읠 받지만, 상위는 하위에 영향을 받지 않도록 구축할 수 있다.

## 실사용 예제

### Propagation.SUPPORTS
```java
@Transactional
public void saveProduct() {
    for (int i = 0; i < 10; i++) {
        try {
            productDao.save(i, "product " + i);
        } catch (RuntimeException ex) {
            // 아무것도 안하기
        }
    }
}

// @Transactional(propagation = Propagation.SUPPORTS)
public void save(int id, String name) {
    if(id > 5) {
        throw new RuntimeException();
    }
    
    String sql = "INSERT INTO test_table VALUES(?, ?)";
    jdbcTemplate.update(sql, id, name);
}
```

안붙일 경우:
Transactional의 aspect() -> saveProduct() -> save() 순으로 실행됨.  
`RuntimeException`을 catch하여 aspect로 정상적으로 도달하게 되며 해당 transaction은 성공적으로 수행된다..

DB 결과:
```java
0, product 0
1, product 1
2, product 2
3, product 3
4, product 4
5, product 5
```
이미 여러 번의 예외가 발생했음에도 불구하고 commit이 수행된다.

붙일 경우:
`propagation = Propagation.SUPPORTS`는 이미 트랜잭션이 존재한다면 해당 트랙잭션을 사용한다.  
하여, 예외를 발생시켰을 때, 해당 트랜잭션이 롤백되도록 만들 수 있으며, catch를 하더라도 이미 롤백으로 표시되어 아무런 정보도 데이터베이스에 저장되지 않는다.  
aspect() -> saveProduct() -> aspect() -> b()

### Propagation.REQUIRES_NEW
결과에 상관없이 호출되는 메서드의 데이터베이스 작업을 데이터베이스에 저장하는 경우 해당 특성을 사용한다.  
예로, 시도된 모든 주식 거래가 감사DB에 기록되어야 한다고 가정했을 때,  
검증 오류, 자금 부족 등의 이유로 거래가 실패했는지 여부와 관계없이 이 정보가 저장되어야 한다면?

감사 메서드에 REQUIRES_NEW 특성을 사용하지 않으면, 감사 레코드는 거래 시도와 함꼐 롤백된다.  
이때, 해당 옵션을 사용하면 초기 트랜잭션의 결과와 관계없이 감사 데이터를 저장할 수 있다.

### Propagation.NEVER
NOT_SUPPORTED와 다르게 일시적으로 멈춘 후 실행시키지 않고 종료와 함꼐 예외를 발생한다.  
예를 들어, 특정 트랜잭션과 함께 이메일을 보내야 한다고 가정하자.  
이메일을 보내는 작업에 트랜잭션이 필요 없으므로 마무리 작업으로 성공적으로 수행되면서 롤백되지 않아야 한다면 이를 사용한다.

만약, NEVER를 사용하지 않을 경우 이메일이 성공적으로 송신되지만 롤백되는 경우가 생길 수 있다.  
즉, 해당 메서드를 누군가가 실수로 정상적인 트랜잭션이 아닌 경우에 호출한다면 예외를 발생시켜준다.
