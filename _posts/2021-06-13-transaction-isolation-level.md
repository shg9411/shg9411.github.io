---
title: "트랜잭션 격리수준 (transaction isolation level)"
excerpt: "트랜잭션과 isolation level에 대해"
classes: wide
---
  
# 트랜잭션이란?  
트랜잭션은 DBMS에서 **데이터를 다루는 논리적인 작업의 단위**이며 하나 이상의 SQL 문장으로 이루어져 있습니다.  
이는 데이터베이스에서 데이터를 다룰 때 장애가 일어나는 경우 복구하는 작업의 단위가 되며,  
여러 작업이 동시에 같은 데이터를 다룰 때 이 작업을 서로 분리하는 단위가 됩니다.  
  
## 트랜잭션의 4가지 특징(ACID)
- 원자성(**A**tomicity) : 트랜잭션에 포함된 작업은 **전부 수행되거나 전부 수행되지 않아야(all or nothing)** 한다.
- 일관성(**C**onsistency) : 트랜잭션을 수행하기 전이나 수행한 후나 데이터베이스는 **항상 일관된 상태를 유지**해야 한다.
  > ex) 계좌이체 : 계좌1과 계좌2의 금액 합이 10만원일 때, 계좌1에서 계좌2로 만원을 이체한 후에도 계좌1과 계좌2의 금액 합은 10만원이 되야 한다.
- 고립성(**I**solation) : **수행 중**인 트랜잭션에 다른 트랜잭션이 끼어들어 변경 중인 **데이터 값을 훼손하는 일이 없어야** 한다.
- 지속성(**D**urability) : 수행을 **성공적으로 완료**한 트랜잭션은 변경한 데이터를 **영구히 저장**해야 한다. 저장된 데이터베이스는 저장 직후 혹은 어느 때나 발생할 수 있는 정전, 장애, 오류에 영향을 받지 않아야 한다.

***  
# transaction isolation level ([참고](https://en.wikipedia.org/wiki/Isolation_(database_systems)))  
하지만 ACID의 원칙을 너무 strict하게 지키면 동시성(Concurrency)에 대한 문제가 생기기 때문에
isolation level을 정하여 동시성을 얻을 수 있는 방법을 제공합니다.  
  
1. READ UNCOMMITTED
2. READ COMMITTED
3. REPEATABLE READ
4. SERIALIZABLE  

SQL 표준에서는 위의 4단계의 격리수준을 정의하고 있습니다. 
번호가 높을수록 고립 수준이 높아지며, 동시성이 떨어집니다.  

|Isolation Level|Dirty read|Nonrepeatable read|Phantom read|
|:---:|:---:|:---:|:---:|
|READ UNCOMMITTED|O|O|O|
|READ COMMITTED|X|O|O|
|REPEATABLE READ|X|X|O|
|SERIALIZABLE|X|X|X|  

## 트랜잭션 동시 실행에 따른 문제는 다음과 같습니다.  
- dirty read(uncommitted dependency)
  * 읽기 작업을 하는 트랜잭션 1이 쓰기 작업을 하는 트랜잭션 2가 작업한 commit되지 않은 중간 데이터를 읽는 경우에 발생합니다.
- Nonrepeatable read
  * 트랜잭션 1이 데이터를 읽고 작업 하는 동안 트랜잭션 2로 인해 트랜잭션 1이 다시 데이터를 읽어올 때, 처음과는 다른 값(value)을 반환합니다.
  > A non-repeatable read occurs when, during the course of a transaction, a row is retrieved twice and the values within the row differ between reads.
- Phantom read
  * 트랜잭션 1이 데이터를 읽고 작업 하는 동안 트랜잭션 2로 인해 트랜잭션 1이 다시 데이터를 읽어올 때, 처음과는 다른 결과(rows)를 반환합니다.
  > A phantom read occurs when, in the course of a transaction, new rows are added or removed by another transaction to the records being read.
  
## READ UNCOMMITTED  
가장 낮은 격리수준입니다. dirty read가 가능하며, 이로 인해 하나의 트랜잭션은 commit이 완료되지 않은 다른 트랜잭션으로 인해 변경된 값을 읽을 수 있습니다.  
postgreSQL에서는 지원하지 않으며 권장하지 않는 격리수준입니다.  
  
## READ COMMITTED
SELECT 문장이 수행되는 동안 해당 데이터에 공유락을 걸어둡니다.  
쿼리가 시작되기 전 commit이 완료된 데이터에 대해서만 값을 읽을 수 있습니다.  
많은 DBMS에서 기본값으로 사용하는 격리 수준입니다.  
    
## REPEATABLE READ
시작 시점의 snapshot 하나를 이용하여 트랜잭션 이후 다른 트랜잭션이 commit되더라도 새로 commit된 데이터는 보이지 않습니다.  
MySQL의 기본값으로 설정되어있는 격리 수준입니다.  
  
## SERIALIZABLE
실행 중인 트랜잭션은 다른 트랜잭션으로부터 완벽하게 분리됩니다.  
가장 높은 격리수준입니다.  
가장 엄격하지만 동시성이 낮아 잘 사용되지 않습니다.
