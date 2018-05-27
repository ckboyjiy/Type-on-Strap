---
layout: post
title: 자바스크립트의 모듈
tags: [javascript, module, namespace]
excerpt_separator: <!--more-->
---
자바스크립트의 규모가 커지면 혼자서 작업하긴 어렵다. 여러 사람들이 만든 코드를 합쳐서 하나의 코드로 구성해야될 필요가 생기게 된다.
여러 소스가 하나로 엮이면 다양한 문제들이 발생할 수 있다.
여기서 우리는 모듈이 필요하게 된다.

모듈은 즉 프로그램과 라이브러리를 모듈 단위로 잘개 나누는 행위를 뜻한다.
<!--more-->

## 자바스크립트의 모듈
### 자바스크립트에서의 모듈이란 무엇인가?

프로그램/라이브러리에서 연관된 객체, 함수, 기타 컴포넌트를 함께 말아넣은 컬렉션으로,
나머지 프로그램/라이브러리의 스코프(범위)와는 분리되어 구성함을 말한다.

### 모듈은 우리에게 어떤 이점을 주는가?
* 코드를 여러 모듈로 **분리**하여 깔끔하게 **구획**하고 **조직화**할 수 있다.
* 전역 스코프를 통해 인터페이스하지 않고 모듈 각자 자신의 스코프를 가지므로, **전역변수**의 오염을 줄일 수 있다.
* 타 프로젝트에서도 똑같은 모듈을 임포트하여 사용할 수 있으므로 코드 **재사용성**이 좋아진다.
* 코드의 범위가 좁아지므로 디버깅에 용이하다.

### private 변수 만들기
우리는 이전에 클래스를 만들어 보았다.
학생 클래스를 생각해보자. 감수성이 풍부한 이 학생은 누구에게나 숨기고 싶은 비밀이 있을 수 있다.
아래 코드를 보자.
```javascript
function Student(name, age, gender, interests) {
    Person.call(this, name, age, gender, interests);
    this.grades = 0; // 학점을 저장하는 변수
}
Student.prototype = Object.create(Person.prototype);
Student.prototype.constructor = Student;
Student.prototype.setGrades = function(score) { // 학점을 부여하는 메서드
    this.grades = score;
}
Student.prototype.getGrades = function() { // 학점을 알려주는 메서드
    return `My grades is ${this.grades} points.`;
}
var student = new Student('ckboyjiy', 33, 'male', ['javascript', 'typescript']);
student.setGrades(44); // 학생은 자신의 점수를 전해 듣는다.
console.log(student.getGrades()); // 학생은 자신의 점수를 알려준다.
console.log(`That student's grades is ${student.grades}.`); // 헉 누군가 나의 점수를 훔쳐보았다.
```
위와 같이 우리가 만든 클래스에 만든 변수 및 메서드는 모두 public 속성과 같다.
누구나 마음대로 열어서 확인하거나 극단적으로는 변경을 할 수도 있다.

클래스에 private 변수를 만들려면 어떻게 해야할까?
우리는 클로저와 즉시 실행 함수 표현(IIFE - Immediately-Invoked Function expression)을 이용하여 private 변수를 흉내낼 수 있다.

#### 어휘적 유효 범위(Lexical Scoping)와 클로저(Closure)
클로저는 함수와 함수가 선언된 어휘적 환경의 조합이라고 한다.
도통 무슨말인지 모르겠다.
함수가 접근할 수 있는 범위(Scope)가 있다.
이 범위는 함수가 실행될 때가 아니라 함수가 정의될 때 결정된다. 이것을 어휘적 유효 범위(Lexical Scoping)이라고 한다.

##### 어휘적 유효 범위(Lexical Scoping)
아래의 코드를 보자.
```javascript
var global = 'Global';
function Outer() { // 외부 함수
    var outer = 'Outer';
    console.log(global);
    function Inner() { // 내부 함수
        console.log(outer);
    }
    Inner();
}
Outer();
console.log(outer); // ReferenceError: outer is not defined
```
위 코드는 두개의 함수가 존재한다.
함수의 범위는 함수가 정의될 때 결정이 된다고 하였다. 첫번째 함수인 Outer부터 보자.

Outer함수는 코드의 최상위, 즉, 전역에서 정의된 함수이다.
그러므로 전역에 선언된 global변수를 접근할 수 있다.
Outer함수 내부에는 Inner 함수가 정의되어 있다.
Inner 함수는 Outer 함수에서 정의되었으므로 Outer 함수의 범위를 가지고 있다.
그러므로 Outer 함수에서 정의한 outer 변수를 참조할 수 있다.

위 예제는 어휘적 유효 범위 지정이 소스 코드 내에서 변수가 선언된 위치를 사용하여 변수의 사용 가능한 위치를 결정한다는 예를 보여준다.
중첩된 함수들은 그들의 외부 유효 범위에서 선언된 변수들을 접근할 권한을 가진다.

##### 클로저(Closure)
어휘적 유효 범위를 바탕으로 우리는 클로저를 구성할 수 있다.
클로저는 위에서 언급한 것처럼 함수와 함수가 선언된 어휘적 유효 범위의 조합이라고 한다.
역시 무슨 말인지 모르겠다.

아래 예제를 보자.
```javascript
function Outer() {
    var outer = 'Outer';
    function Inner() {
        console.log(outer);
    }
    return Inner; // 1.
}
var outer = Outer(); // 2.
outer(); // 3.
```
조금전에 본 어휘적 유효 범위를 설명한 예제와 비슷하지만 몇가지 차이점이 있다.
1. Outer 함수는 내부에 정의된 Inner함수를 리턴하고 있다.
2. Outer 함수를 실행하여 Inner함수를 리턴받아 outer 변수에 저장하고 있다.
3. outer를 실행하면 Inner함수가 실행된다.

Inner 함수가 실행되는 시점은 1번이 아니라 3번시점이다.
다른 프로그래밍 언어는 일반적으로 함수가 실행되는 3번 시점에 접근가능한 지역변수가 정해진다.
하지만 자바스크립트는 어휘적 유효 범위를 가지고 있으므로 정의시점에 정해진다.
즉. 사용가능한 지역변수의 범위는 1번 시점에 정해지고, 실제 함수의 사용은 3번 시점에서 발생된다는 점이다.

Outer 함수에서 선언된 outer 변수는 Outer함수의 범위에서만 접근할 수 있다.
Inner함수의 외부 유효 범위에 Outer 함수가 있으므로 Inner 함수에서도 outer 변수를 접근할 수 있다.
Inner 함수의 범위라하면 <code>Inner() {...여기...}</code>이 안에서 사용가능하다는 뜻이다.
3번 outer()를 실행하는 위치는 전역의 위치이므로 Outer 함수의 외부 범위이므로 outer 변수를 접근할 수 없다.

이것을 이용하여 우리는 private 변수를 흉내내도록 하겠다.
아래 코드를 보자.
```javascript
function StudentClass() {
    var grades = 0;
    function _Student(score) {
        grades = score;
    }
    _Student.prototype.getGrades = function() {
        console.log(grades);
    }
    return _Student; // 1.
}
var A = StudentClass(); // 2.
var a = new A(10); // 3.
a.getGrades(); // 4.
console.log(a.grades); // 5.
```
1. StudentClass 함수에서는 내부 함수인 _Student함수를 리턴한다.
2. StudentClass함수를 실행하면서 A변수에 _Student함수를 리턴받는다.
3. new 연산자를 사용하여 새로운 _Student 객체를 생성한다.
4. getGrades()메서드를 이용하여 내부의 grades 변수값을 출력한다.
5. . 연산자를 이용하여 직접적으로 값을 얻을 수 없어 undefined가 출력된다.

위 코드를 이용해서 private 변수를 흉내낼 수 있었다.
하지만 코드가 아름답지는 않다. 전역변수에는 StudentClass함수도 선언되어 있고, 이 함수의 리턴값인 _Student도 A변수에 의해 참조되고 있다.
우리는 이것을 즉시 실행 함수 표현을 이용해서 조금 깔끔하게 변경할 수 있다.

##### 즉시 실행 함수 표현(IIFE - Immediately-Invoked Function expression)
함수 표현식을 괄호()로 감싸고 그 괄호 뒤에 매개변수를 넘기는 괄호를 붙이면 함수를 정의 즉시 실행되게 할 수 있다.
위에 StudentClass를 즉시 실행 함수 표현으로 바꾸면 아래와 같다.
```javascript
var Student = (function StudentClass() {...})();
```
조금 더 깔끔하게 보이도록 함수 표현식은 익명함수 표현식으로 바꾸도록 하고 함수명도 조금 고쳐보자.
```javascript
var Student = (function() { // 익명 함수식으로 바꿨다.
    var grades = 0;
    function Student(score) { // _를 제거하여 Student로 지칭했다.
        grades = score;
    }
    Student.prototype.getGrades = function() {
        console.log(grades);
    }
    return Student;
})();
var student = new Student(10);
student.getGrades();
console.log(student.grades); // undefined
```
이전 코드보단 만족스럽게 변경된 것 같다. 또한 grades 변수를 직접적으로 참조할 수도 없다.
즉 private 변수를 구현한 것이다.

우리는 클로저와 즉시 실행 함수 표현을 이용하여 클래스 자체의 내부 스코프를 가지고, 전역변수의 오염을 줄였다.
또한 이를 통해 코드를 분리하여 조직화할 수 있었다.

이것 외에 다양한 디자인 패턴들과 결합하여 모듈을 설계해 나갈 수 있을 것이다.
