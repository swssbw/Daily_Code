## 정의

**종료된(생명 주기가 끝난) 외부 함수의 변수를 참조하는 내부 함수를 클로저 라고 한다.**

꼭 항상 함수를 지칭하지는 않고.. **내부함수가 외부함수의 컨텍스트에 접근할 수 있는것** 을 가리킨다.

(내부함수를 지칭하며, 그 함수가 선언된 환경까지 기록)

함수가 호출되는 환경과 별개로, 기존에 선언되어있던 환경을 기준으로 변수를 조회한다.

자바스크립트만의 고유한 개념은 아니며, 여러 함수형 프로그래밍 언어에서 공통적으로 발견되는 특성이다.

### 장점과 단점

1. 클로저는 함수내의 지역변수를 감추는 은폐의 역할을 가능하게 한다. (다른 언어의 private 키워드와 비슷한 역할을 함) 

자바스크립트는 private 변수, 즉 외부에서 접근할 수 없는 변수 형태를 선언할 수 없는데, 동일한 기능을 클로저를 통해 구현할 수 있다. 

2. 무분별한 전역변수의 사용을 억제하고 발생 가능한 변수 간의 충돌을 막을 수 있으며, 이는 곧 이벤트 및 애니메이션의 효과적인 함수 사용을 가능하게 한다. 클로저를 활용하여 개발한다면 복잡한 코드를 쉽고 간결하고 작성할 수 있다.

어떻게 사용하느냐에 따라 달라지겠지만 기본적으로 메모리와 퍼포먼스 저하라는 문제가 발생할 수도 있다.

## 예시

### 기본

*에서 outer가 종료되었는데도 **에서 a에 접근할 수 있다!

```jsx
function outer() {
  const a =  "aaa";
  console.log("outer", a);
  return function () {
    console.log("inner", a);
  }
}

const getA = outer(); // outer aaa *
getA(); // inner aaa **
```

### 심화 1 - 자바스크립트에서 private 속성 따라하기

```jsx
function fruit(emoji){
  return {
      get : function (){
          return emoji;
      },
      set : function(_emoji){
          emoji = _emoji
      }
  }
}

let 사과 = fruit('🍎');
let 바나나 = fruit('🍌');

1) console.log(사과.get()); // 🍎
2) console.log(바나나.get()); // 🍌
3) 사과.set('🍏');
4) console.log(사과.get()); // 🍏
5) console.log(바나나.get()); // 🍌
```

1. 예제는 함수의 리턴으로 객체를 반환하고 있으며, 해당 객체에는 메소드 get과 set을 가진다. 이 메소드들은 외부함수인 fruit의 인자로 전달된 지역변수 name을 사용한다. (클로저는 객체의 메소드에서도 사용할 수 있다.)
2. 동일한 함수 내부에서 만들어진 메소드들은 외부함수의 지역변수를 공유한다.  `3)` 에서 set해준 🍏를 `4)`에서도 확인할 수 있다. get과 set이 emoji 값을 공유하고 있다는 뜻!
3. 똑같은 외부함수 fruit을 공유하고 있는 사과, 바나나 의 get 결과는 서로 다르다. 
왜냐, 외부함수가 실행될 때마다 새로운 지역변수를 포함하는 클로저가 생성되기 때문이다. 
사과, 바나나는 완전 독립된 (다른) 클로저를 가진다.
4.  emoji의 값을 읽고 쓰기 위해서는 fruit 메소드의 리턴으로 만들어진 객체를 사용해야한다.  
(fruit 의 지역변수 emoji는 fruit 내부에서 정의된 객체(fruit이 리턴하는 객체)의 메소드에서만 접근 할 수 있다)  
JavaScript는 private 속성을 지원하지 않는데, 클로저의 특성을 이용해 private한 속성을 사용할 수 있게 된다.

### 심화 2 - 반복문에서 클로저 사용할 시 문제점

```jsx
let arr = [];

for(**var** i = 0; i < 3 ; i++) {
  arr[i] = function inner() {
    console.log(i);
  }
}

for(let j = 0; j < 3 ; j++) {
  arr[j]();
}

// 3
// 3
// 3
```

1>2>3 을 순서대로 출력하고 싶었는데 실제로는 3>3>3이 출력된다. 왜일까?

inner 는 클로저로, 언제 어디서 호출되던 항상 상위스코프인 for문에서 i값을 찾는다. 

이때 찾는 시점은 이미 for문이 끝난 상태기 때문에, 반복문이 끝난 후의 값인 i = 3만 출력된다. 

**반복문에서 클로저 사용 시, 클로저는 반복문이 끝날 시점의 값을 가진다.**

의도한 대로 동작하게 하려면

1. 새로운 스코프를 추가하여 반복마다 따로 값을 저장하게 하기
2. ES6에서 추가된 블록스코프 이용하기

### 해결 1 - 새로운 스코프를 추가해 반복마다 따로 값을 저장하게 하기

```jsx
let arr = [];

for(var i = 0; i < 3 ; i++) {
  (function (_i) {
    arr[_i] = function() {
      console.log(_i);
    }
  })(i);
}

for(var j = 0; j < 3 ; j++) {
  arr[j]();
}

// 0
// 1
// 2
```

### 해결 2 - ES6에서 추가된 블록스코프 이용하기

i 는 for문을 돌 때 마다 중괄호가 형성된다.. 즉 새로운 스코프가 생긴다. 

```jsx
let arr = [];

for(**let** i = 0; i < 3 ; i++) {
  arr[i] = function inner() {
    console.log(i);
  }
}

for(let j = 0; j < 3 ; j++) {
  arr[j]();
}

// 0
// 1
// 2
```