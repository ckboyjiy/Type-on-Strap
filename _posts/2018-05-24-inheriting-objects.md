---
layout: post
title: 자바스크립트의 프로토타입 체인과 객체 상속
tags: [javascript, object, prototype, inherit]
excerpt_separator: <!--more-->
---

자바스크립트의 프로토타입 체인을 활용하여 상속을 구현하는 방법에 대해 알아보자.
<!--more-->

## 객체지향 프로그래밍 - 기초
우리는 지난번에 Person이라는 객체를 만들어 보았다. 이것은 사실상 객체지향언어의 클래스와 유사하다.

우리는 이제 Person 클래스를 상속받아 Student와 Teacher를 만들어보자.

### 그럴듯한 Peron 클래스
지난번에 만든 Person 객체(이제부터 클래스라고 하자)는 구조적으로는 많이 좋아졌지만 세부 구성은 형편이 없었다.
먼저 그럴듯한 구성으로 조금 보강을 해보자.

이름 외에 나이, 성별 그리고 취미들을 추가해보자.

```javascript
function Person(name, age, gender, interests) {
    this.name = name;
    this.age = age;
    this.gender = gender;
    this.interests = interests;
}
Person.prototype.greeting = function() {
    console.log("Hi! I'm " + this.name + ".");
}
var ckboy = Object.create(Person.prototype, {
    name: {value: 'ckboyjiy', enumerable: true},
    age: {value: 33, enumerable: true},
    gender: {value: 'male', enumerable: true},
    interests: {value: ['javascript', 'typescript'], enumerable: true}
});
// var ckboy = new Person('ckboyjiy', 33, 'male', ['javascript', 'typescript']);
ckboy.greeting();
console.log(ckboy); // Person { name: 'ckboyjiy', age: 33, gender: 'male', interests: ['javascript', 'typescript'] }
```

사람 클래스가 완성되었다.
이제 이 사람(Person) 클래스를 가지고 선생(Teacher) 클래스와 학생(Student) 클래스를 만들어보자.

### Teacher 클래스 만들기
우리가 만든 Person 클래스를 기본으로 선생님에 특화된 속성을 추가해보자.
선생님은 각기 자신이 맞은 과목이 존재한다. 우리는 과목(subject) 속성을 추가해보자.

```javascript
// Person 생략
function Teacher(name, age, gender, interests, subject) {
    Person.call(this, name, age, gender, interests); // Person 클래스의 생성자 함수를 호출하여 속성을 상속
    this.subject = subject;
}

var teacher = new Teacher('ckboyjiy', 33, 'male', ['javascript', 'typescript'], 'mathematics');
console.log(teacher); // Teacher { name: 'ckboyjiy', age: 33, gender: 'male', interests: ['javascript', 'typescript'], subject: 'mathematics' }
console.log(Object.getPrototypeOf(teacher)); // {}
teacher.greeting(); // typeError: teacher.greeting is not a function
```

생각보다 간단하게 Person 클래스로부터 정상적으로 상속받은 것처럼 보인다. 물론 현재는 속성만 상속받은 것이다.
console.log로 출력된 것과 같이 teacher의 prototype에 greeting 함수가 없고, 호출해보아도 에러만 발생할 뿐이다.

#### 프로토타입 상속하기(연결하기)
greeting메서드도 사용하기 위해서는 별도로 Person 클래스의 프로토타입에 정의한 함수도 상속시켜줘야 한다.

##### 프로토타입 복사
우리는 이미 원하는 프로토타입을 가진 객체를 만드는 법을 알고 있다. 바로 Object.create() 함수이다.

```javascript
function Teacher(name, age, gender, interests, subject) {
    Person.call(this, name, age, gender, interests);
    this.subject = subject;
}
Teacher.prototype = Object.create(Person.prototype); // 프로토타입 상속

var teacher = new Teacher('ckboyjiy', 33, 'male', ['javascript', 'typescript'], 'mathematics');
console.log(teacher); // Person { name: 'ckboyjiy', age: 33, gender: 'male', interests: ['javascript', 'typescript'], subject: 'mathematics' }
console.log(Object.getPrototypeOf(Teacher.prototype)); // Person { greeting: [Function] }
teacher.greeting(); // Hi! I'm ckboyjiy.
```
잘 작동되는 것 같다. 인스턴스의 속성도 정상적으로 잘 초기화되어 있고, 기존에 호출이 안되던 greeting함수도 잘 호출된다.
그런데 이상한 것이 보인다. <code>console.log(teacher);</code>의 결과가 바로 콘솔로그에 전에는 Teacher {...}로 나오던 것이 Person {...}으로 나오고 있다.
이것은 무엇인가?!

분명히 new Teacher(...)로 인스턴스를 만들었는데 Person 객체로 만들었단다.
new 연산자는 어떻게 동작하는 것인지 알아보자.

#### new 연산자
```javascript
var teacher = new Teacher('ckboyjiy', 33, 'male', ['javascript', 'typescript'], 'mathematics');
```

1. Teacher.prototype(Person.prototype을 프로토타입으로 가지고 있는 객체)을 상속하는 새로운 객체(A라고 하자)를 하나 생성한다.
2. A의 this를 가지고 생성자 함수 Teacher()가 호출된다.
3. 생성자 함수에서 리턴된 객체는 new 연산자의 결과가 된다.
4. 생성자 함수에 리턴되는 값이 없다면? A 객체가 리턴된다. (일반적으로 리턴값이 없다!)

1번에서 이미 Person.prototype을 기반으로 만든 Person 객체가 생성되었고, 생성자 함수의 역할은 단지 Person 객체의 this의 값을 초기화하는 역할만 해줄 뿐이다.
즉, 정말로 우리가 만든 객체는 Teacher 인스턴스가 아니라 Person 인스턴스인 것이다.

#### 프로토타입의 생성자 변경
prototype에는 constructor라는 속성이 있다. 모든 객체는 자신의 prototype으로부터 constructor 속성을 상속한다.
new 연산자 설명의 1번의 새로운 객체를 생성할 때 참조하는 대상이 바로 이 constructor이다.

우리는 이제 프로토타입의 생성자를 변경해보자.
```javascript
function Teacher(name, age, gender, interests, subject) {
    Person.call(this, name, age, gender, interests);
    this.subject = subject;
}
Teacher.prototype = Object.create(Person.prototype);
Teacher.prototype.constructor = Teacher; // 프로토타입의 생성자 변경

var teacher = new Teacher('ckboyjiy', 33, 'male', ['javascript', 'typescript'], 'mathematics');
console.log(teacher); // Teacher { name: 'ckboyjiy', age: 33, gender: 'male', interests: ['javascript', 'typescript'], subject: 'mathematics' }
console.log(teacher.constructor); // [Function: Teacher]
console.log(Teacher.prototype.constructor); // [Function: Teacher]
```

이제 모든 것이 잘 맞는 것 같다.

참고로 인스턴스의 생성자를 확인하는 방법은 <code>인스턴스.constructor</code>이다.

객체의 프로토타입의 생성자를 확인하는 방법은 <code>객체.prototype.constructor</code>이다.

### 메서드 추가 및 변경
Teacher 클래스를 무사히 잘 만든 것 같다.
하지만 명색이 선생님인데 인사문구가 너무 짧은 것 같다. 이렇게 변경해보자.

**'Hello. My name is Mr.(또는 Mrs.) 이름, and I teach 과목.'**

그리고 과목을 변경하는 메서드도 추가해보자.

메서드의 추가나 변경은 동일하고 매우 간단하다.
```javascript
function Teacher(name, age, gender, interests, subject) {
    Person.call(this, name, age, gender, interests);
    this.subject = subject;
}

Teacher.prototype = Object.create(Person.prototype);
Teacher.prototype.constructor = Teacher;
Teacher.prototype.greeting = function() { // 메서드 변경
    var prefix = this.gender === 'male' ? 'Mr. ' : 'Mrs. ';
    console.log("Hello. My name is " + prefix + this.name + ", and I teach " + this.subject + ".");
}
Teacher.prototype.changeSubject = function(subject) { // 메서드 추가
    this.subject = subject;
}

var teacher = new Teacher('ckboyjiy', 33, 'male', ['javascript', 'typescript'], 'mathematics');
teacher.changeSubject('history');
teacher.greeting(); // Hello. My name is Mr. ckboyjiy, and I teach history.
```

