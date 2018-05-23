---
layout: post
title: 자바스크립트의 객체 생성 및 상속
tags: [javascript, class, inherit]
excerpt_separator: <!--more-->
---

자바스크립트의 객체 생성 방법과 프로토타입 체인을 활용하여 상속을 구현하는 방법에 대해 알아보자.
<!--more-->
## 객체 생성
자바스크립트에서 객체를 생성하는 방법은 여러가지가 있다.
* 객체 리터럴을 선언
* 생성자 함수를 사용
* Object 생성자 사용
* Object.create() 함수를 사용

먼저 객체 리터럴을 이용하여 생성하는 방법을 알아보자

```javascript
var person = {};
```

일반적으로 객체는 데이터와 함수들의 집합으로 이뤄져 있다. 객체에 데이터를 추가해보자.

```javascript
var person = { name: 'ckboyjiy'}
```

객체에 함수를 추가해보자.

```javascript
var person = {
    name: 'ckboyjiy',
    greeting: function() {
            console.log("Hi! I'm " + this.name + ".");
        }
    };
```
객체가 생성된 이후에도 추가가 가능하다.
```javascript
var person = {};
person.name = 'ckboyjiy';
person.greeting = function() {
    console.log("Hi! I'm " + this.name + ".");
};
```

### this는 무엇인가?
greeting 함수에서 사용되는 this는 C나 Java를 알고 있는 분들이 항상 해깔려하는 부분이다.
일단 이 시점에서는 단순하게 해당 코드(함수)를 가지고 있는 객체라고만 알아두자.

## 객체지향 프로그래밍 - 기초
우리는 위에서 person이라는 객체를 만들어 보았다. 학교를 예로 들자면, 학교 안에서 사람을 세분화 해본다면 학생과 선생님으로 나눌 수 있다.
객체지향 프로그래밍에서는 특정 클래스를 기반으로 새로운 클래스를 만들 수 있다. --Child 클래스는 Parent 클래스를 상속받아서 만들어진다 --
우리는 Person이라는 사람 클래스를 만들고 Person 클래스를 상속받아 Student와 Teacher를 만들어보자.

### 객체와 클래스
위에서 우리가 만든 person 객체는 우리가 알고 있는 클래스와 동일하지 않다.
객체는 클래스의 인스턴스다. 우리가 만든 person 객체를 여러개 생성하려고 한다면 에러가 발생한다.

```javascript
var person = {};
var student = new person(); //TypeError: object is not function
```

오브젝트가 함수가 아니라서 에러가 발생한다. 즉 new 키워드는 함수만 사용할 수 있다.
### 생성자와 객체 인스턴스
자바스크립트는 새로운 인스턴스가 필요할 때마다 객체를 재구성해야 한다.
객체와 그 기능을 정의하기 위해 **생성자 함수**라는 특별한 함수가 필요하다.
그리고 new 키워드를 사용하여 생성자 함수를 호출하여 인스턴스를 만들어야 한다.

#### 형편없는 생성자 함수 만들기
아래 코드를 사용하여 생성자 함수 및 생성자 함수를 통한 새로운 인스턴스를 만들어보자.
```javascript
function Person() {
    this.name = 'ckboyjiy';
    this.greeting = function() {
        console.log("Hi! I'm " + this.name + ".");
    }
}

var person1 = new Person();
```
된다! 새로운 인스턴스가 생성이 된다!

근데 아무리 많은 객체를 만들어도 다 이름이 'ckboyjiy'다. 생성자 함수를 사용할 때 파라미터를 넘겨서 초기화에 사용할 수 있다.
```javascript
function Person(name) {
    this.name = name;
    this.greeting = function() {
        console.log("Hi! I'm " + this.name + ".");
    }
}

var ckboy = new Person('ckboyjiy');
ckboy.greeting();
var ckgirl = new Person('ckgirl');
ckgirl.greeting();
```
**생성자 함수**는 반환값이 없으며 단순히 멤버변수와 멤버함수를 정의한다.
그리고 this를 사용하여 생성된 인스턴스마다 자신의 멤버변수(멤버함수)를 바라보도록 한다.
마지막으로 가독성을 위해 생성자 함수 이름의 첫글자는 대문자를 사용하도록 하자.

#### 완벽한 생성자 함수 만들기
위에서 만든 객체를 한번 콘솔로 찍어보자.
```javascript
console.log(ckboy);
> Person { name: 'ckboyjiy', greeting: [Function] }
console.log(ckgirl);
> Person { name: 'ckgirl', greeting: [Function] }
```
새로운 인스턴스가 만들어 질 때마다 name 및 greeting을 매번 정의하게 된다.
불필요한 greeting 함수가 매번 정의되고 있는 것이다. 
