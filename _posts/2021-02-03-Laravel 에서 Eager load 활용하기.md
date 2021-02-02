---
layout: post
title:  "Laravel 에서 Eager load 활용하기"
date:   2021-02-03 00:40:12
---

## Eager load ?

---

ORM (Object Relational Mapping)은 데이터베이스 작업을 놀랍도록 간단하게 만듭니다. 객체 지향 방식으로 데이터베이스 관계를 정의하면 관련 모델 데이터를 쉽게 쿼리 할 수 있지만 개발자는 기본 데이터베이스 호출에주의를 기울이지 않을 수 있습니다.

ORM으로 작업 할 때 데이터 가져 오기 /로드는 eager와 lazy의 두 가지 유형으로 분류 할 수 있습니다.

At a basic level, ORMs “lazy” load-related model data
기본 수준에서 ORM은 "지연"로드 관련 모델 데이터입니다

$books = App\Book::with(['author', 'publisher'])->get();

$books = App\Book::with('author:id,name')->get();

vs lazy load

n+1 문제

++ 
 index 문제

<br><br><br>
