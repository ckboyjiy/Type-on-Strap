---
layout: post
title: 자바스크립트의 객체 생성
tags: [javascript, object, create]
excerpt_separator: <!--more-->
---

자바스크립트의 객체 생성 방법에 대해 알아보자.
<!--more-->
## 객체 생성
자바스크립트에서 객체를 생성하는 방법은 여러가지가 있다.
* 객체 리터럴을 선언
* 생성자 함수를 사용
* Object 생성자 사용 ( var instance = new Object(); // 자세한 내용은 생략한다. )
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

> #### this는 무엇인가?
> greeting 함수에서 사용되는 this는 C나 Java를 알고 있는 분들이 항상 해깔려하는 부분이다.
일단 이 시점에서는 단순하게 해당 코드(함수)를 가지고 있는 객체라고만 알아두자.

### 객체와 클래스
위에서 우리가 만든 person 객체는 우리가 알고 있는 클래스와 동일하지 않다.
우리가 만든 person 객체를 여러개 생성하려고 한다면 에러가 발생한다.

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
**생성자 함수**는 반환값이 없으며 단순히 멤버변수(속성)와 멤버함수(함수)를 정의한다.
그리고 this를 사용하여 생성된 인스턴스마다 자신의 멤버변수(또는 멤버함수)를 바라보도록 한다.
마지막으로 가독성을 위해 생성자 함수 이름의 첫글자는 대문자를 사용하도록 하자.

#### 완벽한 생성자 함수 만들기
위에서 만든 객체를 한번 콘솔로 찍어보자.
```javascript
console.log(ckboy); // Person { name: 'ckboyjiy', greeting: [Function] }
console.log(ckgirl); // Person { name: 'ckgirl', greeting: [Function] }
console.log(ckboy.greeting === ckgirl.greeting); // false
```
새로운 인스턴스가 만들어 질 때마다 name 및 greeting을 매번 정의하게 된다.
불필요한 greeting 함수가 매번 정의되고 있는 것이다. 이는 결국 메모리의 효율성이 좋지 않다는 것이다.

그렇다 우리는 형편없는 인스턴스를 만들었던 것이다.

> #### **프로토타입 기반 언어**
> 자바스크립트의 객체들은 각각 프로토타입 객체가 존재한다. 이 객체는 메서드 및 속성을 상속하는 템플릿 객체 역할을 한다.
> #### **프로토타입 체인**
> 객체는 자신의 프로토타입이 없을 때까지 반복적으로 참조한다. (예 : Student() <- Person() <- {} <- undefined)
>
> 자세한 내용은 추후 별도로 포스팅하겠음.

위에 잠시 언급한 프로토타입 체인을 이용하여 프로토타입 상속을 구현하려고 한다.

먼저 기존에 만든 Person 함수의 멤버변수와 멤버함수를 분리시켜야 한다.
아래와 같이 멤버변수는 생성자 함수에 놔두고, 멤버함수는 프로토타입에 정의한다.

```javascript
function Person(name) {
    this.name = name;
}
Person.prototype.greeting = function() {
    console.log("Hi! I'm " + this.name + ".");
}
var ckboy = new Person('ckboyjiy');
var ckgirl = new Person('ckgirl');
console.log(ckboy); // Person { name: 'ckboyjiy' }
console.log(ckgirl); // Person { name: 'ckgirl' }
ckboy.greeting(); // Hi! I'm ckboyjiy.
ckgirl.greeting(); // Hi! I'm ckgirl.
console.log(Object.getPrototypeOf(ckboy)); // Person { greeting: [Function] }
console.log(Object.getPrototypeOf(ckgirl)); // Person { greeting: [Function] }
console.log(Object.getPrototypeOf(ckboy) === Object.getPrototypeOf(ckgirl)); // true
console.log(ckboy.greeting === ckgirl.greeting); // true
```

* 콘솔로그에 표시되는 것처럼 ckboy와 ckgirl은 name이라는 멤버변수(속성)만 가지고 있다.
* 하지만 greeting()함수도 정상적으로 호출되는 것을 확인할 수 있다.
* 각 인스턴스의 prototype에 greeting 함수가 존재하며 두 인스턴스는 동일한 prototype객체를 참조하는 것을 확인할 수 있다.
* 그리고 각 인스턴스에서 사용한 greeting 함수가 동일한 것도 확인할 수 있다.

위처럼 생성자 함수와 프로토타입을 이용하면 클래스와 유사하게 자바스크립트 객체를 생성할 수 있다.

new 키워드를 이용한 생성자 함수 호출을 하지 않고 객체를 생성하는 또다른 방법을 소개하겠다.
바로 블로그 처음에 언급한 Object.create() 함수이다.

#### Object.create()
Object.create()함수는 ES5에 도입된 구문이다.
사용법은 아래와 같다.
```javascript
Object.create(proto[, propertiesObject])
```
자세한 API는 [MDN API](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create) 에서 참고

```javascript
function Person(name) {
    this.name = name;
}
Person.prototype.greeting = function() {
    console.log("Hi! I'm " + this.name + ".");
}
var ckboy = Object.create(Person.prototype, {
    name: {value: 'ckboyjiy'}
});
ckboy.greeting();
var ckgirl = new Person('ckgirl');
ckgirl.greeting();
console.log(ckboy); // Person {}
console.log(ckgirl); // Person { name: 'ckgirl' }
console.log(Object.getPrototypeOf(ckboy) === Object.getPrototypeOf(ckgirl)); // true
```
ckboy변수는 Object.create()를 사용하여 객체를 생성하였다.
이는 이전 **생성자 함수를 사용하지 않고** 객체를 사용하기 때문에 명시적으로 <code>name: {value: 'ckboyjiy'}</code>와 같이 name 필드를 정의했다.

특이한 점은 greeting()함수 호출에서 "Hi! I'm ckboyjiy." 문구가 정상적으로 나오는 걸 봐서는 name값이 있다는 것인데..
console.log(ckboy)로 찍어보면 Person {} 값이 비어 있다.
이것은 enumerable 의 기본값이 false인데 이것은 프로퍼티를 가본적으로 숨긴다는 속성이다. ckgirl 처럼 값이 표시되길 원하면 아래와 같이 true로 지정해주면 된다.

```javascript
function Person(name) {
    this.name = name;
}
Person.prototype.greeting = function() {
    console.log("Hi! I'm " + this.name + ".");
}
var ckboy = Object.create(Person.prototype, {
    name: {value: 'ckboyjiy', enumerable: true}
});
ckboy.greeting();
var ckgirl = new Person('ckgirl');
ckgirl.greeting();
console.log(ckboy); // Person { name: 'ckboyjiy' }
console.log(ckgirl); // Person { name: 'ckgirl' }
console.log(Object.getPrototypeOf(ckboy) === Object.getPrototypeOf(ckgirl)); // true
```