# var, let, const
`var`, `let`, `const`는 변수 선언시 사용할 수 있는 키워드로, 각각 특성을 가지고 있습니다.  
`var`와 `let`은 변수로써 값을 바꿀 수 있는 특성을 가지고 있습니다.  
`const`는 상수로써 처음에 값이 지정되면 바꿀 수 없는 특성을 가지고 있습니다.

## 1. 특징
### var
```javascript
var a = 0;
console.log(a); // 0
a = 1;
console.log(a); // 1
var a = 2;
console.log(a); // 2
```
`var`의 특징은 변수로써 재선언과 재할당이 가능합니다.

### let
```javascript
let a = 0;
console.log(a); // 0
a = 1;
console.log(a); // 1
let a = 1; // SyntaxError: Identifier 'a' has already been declared
```
`let`는 ES6에 추가된 키워드로 재할당은 가능하지만, 재선언은 불가능합니다.

### const
```javascript
const a = 0;
console.log(a); // 0
a = 1; // TypeError: Assignment to constant variable.
```
`const`는 ES6에 추가된 키워드로 재선언과 재할당이 불가능합니다.  
다만 객체의 경우엔 요소의 값을 변경하는 것은 가능하므로, `const` 키워드가 반드시 불변성을 가지는 것은 아닙니다.

## 2. Scope
스코프는 변수에 접근할 수 있는 '범위'를 지칭하는 것으로, JavaScript에선 크게 전역과 지역으로 나뉘어집니다. `var`가 다른 두 키워드와 차별되는 점은 이 스코프의 범위입니다.
### var
```javascript
var variable = 0;
(function varTest() {
	var variable = 1;
	if(true) {
		var variable = 2;
		console.log("varTest", "in", variable); // varTest in 2
	}
	console.log("varTest", "out", variable); // varTest out 2
})();
console.log("varTest", "global", variable); // varTest global 0
```
`var`는 함수 스코프의 특성을 가지고 있습니다. 함수 내에서 선언된 `variable`변수는 오직 함수 내에서만 접근할 수 있습니다.  
위의 예제를 보면 전역 스코프의 `variable`는 `0`으로 선언되어있고, 함수 스코프의 `variable`는 `1`로 선언되어 있습니다.  
두 변수의 스코프가 다르기 때문에 전역 스코프의 값이 `1`이 되지 않았습니다.  
반면 `if` 블록에 선언된 `variable`는 함수 스코프에 포함되기 때문에 `variable = 2`로 재정의됩니다. 따라서 `console.log`에서 `2`가 출력됩니다.  
그리고 함수 바깥의 `console.log`에선 전역 스코프 `variable`를 인자로 받았기 때문에 `0`이 출력됩니다.

### let
```javascript
let letX = 0;
(function letTest() {
	let letX = 1;
	if(true) {
		let letX = 2;
		console.log("letTest", "in", letX); // letTest in 2
	}
	console.log("letTest", "out", letX); // letTest out 1
})();
console.log("letTest", "global", letX); // letTest global 0
```
`let`은 블록 스코프의 특성을 가지고 있습니다. 블록 내에서 선언된 `letX`변수는 오직 블록 내에서만 접근할 수 있습니다.  
위의 예제에서 전역으로 선언된 `letX`는 `0`으로, 함수 내의 `letX`은 `1`로 선언되어 있습니다.  
`let`의 블록 스코프 특성에 따라 두 변수는 별개의 것으로 취급되며, 전역 스코프의 `letX`는 `1`이 아닌 `0`입니다.  
한편 `if`블록은 함수 블록과 별개이므로, `if`블록의 `let letX=2`는 함수 블록의 `letX`를 `2`로 변경되지 않고, 새롭게 `letX`가 2로 선언됩니다.  
함수 바깥의 `console.log`에선 전역 스코프의 `letX`를 인자로 받아 `0`이 출력됩니다.

### const
```javascript
const constant = 0;
(function constTest() {
	const constant = 1;
	if(true) {
		const constant = 2;
		console.log("constTest", "in", constant); // constTest in 2
	}
	console.log("constTest", "out", constant); // constTest out 1
})();
console.log("constTest", "global", constant); // constTest global 0
```
`const`는 let과 마찬가지로 블록 레벨 스코프의 특성을 가지고 있습니다. 블록 내에 선언된 `constant`변수는 오직 블록 내에서만 접근할 수 있습니다.
위의 예제에서 전역으로 선언된 `constant`는 `0`으로, 함수 내의 `constant=1`에 의해 재할당 되는 것이 아닌 각각 독립된 변수로 취급됩니다.  
`if` 블록의 `constant` 역시 함수 블록의 `constant`와 다른 블록 스코프를 가지고 있기 때문에 각각 독립된 변수로 취급됩니다.
따라서 각 스코프에서 호출되는 `console.log`는 각각 다른 값을 출력합니다.

## 3. Hoisting
호이스팅(Hoisting)이란 들어올리다/끌어올리다의 뜻을 가진 단어로, JavaScript에선 스코프의 최상단으로 **선언(Declaration)이 끌어올려지는 현상**을 말합니다.

### Execution Context
호이스팅은 JavaScript에서 **실행 컨텍스트(Execution Context)** 의 생성과정에서 일어나며, JavaScript의 **모든 선언** 은 이때 호이스팅 됩니다.  
실행 컨텍스트의 생성과정에서 변수는 선언단계 -> 초기화 단계 -> 할당 단계를 거치며 생성됩니다.

### var
```javascript
console.log(vName); // undefined
var vName = '1';
console.log(vName); // 1
/*-----------------------------*/
var vName = undefined;
console.log(vName);
vName = '1';
console.log(vName);
```
`var`의 선언은 호이스팅에 의해 최상단으로 끌어올려지며 `undefined`로 초기화 됩니다.  
따라서 첫번째 `console.log`에선 `undefined`가 출력됩니다.  
이후 `vName`은 `1`로 초기화 되어 `console.log`엔 `1`이 출력됩니다.
`var`는 선언단계와 초기화단계가 한번에 이루어지기 때문에 할당이 되기 전까지 `undefined`의 값을 가집니다.

### let
```javascript
let foo = 1;
{
  console.log(foo); // ReferenceError: Cannot access 'foo' before initialization
  let foo = 2;
}
```
`let`의 선언은 호이스팅에 의해 최상단으로 끌어올려집니다. 하지만 `let`은 `var`와 달리 선언단계와 초기화단계가 분리되어있습니다.  
따라서 `foo = 1`이 실행되기 전에 `foo`이 호출되고 여기엔 초기화가 되지 않았기 때문에 `Cannot access 'foo' before initialization`(foo는 초기화 이전에 접근할 수 없다) 오류를 출력합니다.  
선언이 호이스팅되고 초기화가 될때까지의 시점을 일시적 사각지대(Temporal Dead Zone, TDZ)라 부릅니다.

### const
```javascript
const bar = 1;
{
  console.log(bar); // ReferenceError: Cannot access 'bar' before initialization
  const bar = 2;
}
```
`const`의 선언은 `let`의 선언과 동일하게 진행됩니다. 선언만 호이스팅되며, 그전에 값을 호출하면 `not defined` 오류를 출력합니다.  
이는 `const` 역시 `let`처럼 선언 단계와 초기화 단계가 분리되어있기 때문입니다.

### 스코프와 호이스팅
```javascript
let a = 0;
{
  var a = 0; // SyntaxError: Identifier 'a' has already been declared
}
```
`let`는 블록 스코프를 가지고, `var`는 함수 스코프를 가지고 있기 때문에 `let`으로 변수를 선언 후 블록에서 `var`를 선언할 경우,  
`var`는 스코프 최상단으로 호이스팅되어 중복 선언 오류를 출력합니다.

## 4. window
### var
```javascript
var var1 = 0; // this.var1
function fooVar1() {
	var var1 = 1;
	console.log(var1); // 1
	console.log(this.var1); // 0 | strict mode에선 TypeError: Cannot read properties of undefined
	console.log(var1 === this.var1) // false
}
console.log(window.var1); // 0
fooVar1()
console.log(var1 === this.var1) // true
```
전역으로 선언된 `var` 변수는 `window` 객체에 자동으로 할당됩니다. 이는 `var`가 가지는 특징입니다.  
`var`의 스코프는 함수 스코프기 떄문에 지역으로 선언된 `var` 변수는 `window`에 할당되지 않습니다.

```javascript
console.log(isNaN) // ƒ isNaN() { [native code] }
var isNaN = 0;
console.log(isNaN) // 0
isNaN(0) // TypeError: isNaN is not a function
```
이처럼 `window` 객체를 잘못 수정할 우려가 있기 떄문에,  
`let`와 `const`의 사용을 권고하는 것으로 보입니다.

### let, const
```javascript
let let1 = 0;
function fooLet1() {
	let let1 = 1;
	console.log(let1) // 1
	console.log(this.let1); // undefined
}
console.log(window.let1); // undefined
fooLet1()
console.log(let1 === this.let1) // false
```
전역으로 선언된 `let`과 `const`는 `window` 객체에 추가되지 않습니다.

## 5. 암묵적인 전역변수 선언
```javascript
var x = 0;
function a() {
	var y = 1;

	console.log(x, y); // 0 1

	function b() {
		x = 3; // window.x
		y = 4; // local y
		z = 5; // window.z | strict mode에선 ReferenceError: z is not defined
	}
	b();

	console.log(x, y, z); // 3 4 5
}

a();
console.log(x, z); // 3 5
console.log(typeof y) // undefined
```
`var`, `let`, `const`와 같은 키워드 없이 변수를 할당하는 경우 자동으로 전역변수로 정의됩니다.  
단, 권장되지 않는 구문으로 사용시 주의해야합니다.


## 참고자료
* [MDN Web Docs : var](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/var)
* [MDN Web Docs : let](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/let)
* [MDN Web Docs : const](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/const)
* [[JavaScript] 호이스팅(Hoisting)이란?](https://hanamon.kr/javascript-%ED%98%B8%EC%9D%B4%EC%8A%A4%ED%8C%85%EC%9D%B4%EB%9E%80-hoisting/)
* [var, let, const 차이점](https://velog.io/@bathingape/JavaScript-var-let-const-차이점)
* [var, let, const의 차이 ⏤ 변수 선언 및 할당, 호이스팅, 스코프](https://www.howdy-mj.me/javascript/var-let-const/)
* [자바스크립트 개발자라면 알아야 할 33가지 개념 #6 함수와 블록 스코프 (번역)](https://velog.io/@jakeseo_me/자바스크립트-개발자라면-알아야-할-33가지-개념-6-함수와-블록-스코프-번역-dijuhrub1x)
* [실행 컨텍스트와 자바스크립트의 동작 원리](https://poiemaweb.com/js-execution-context)
