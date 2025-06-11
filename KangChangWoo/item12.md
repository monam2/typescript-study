# Item 12

## Item.12 함수 표현식에 타입 적용하기

*요약*

- 매개변수나 반환 값에 타입을 명시하기보다는 함수 표현식 전체에 타입 구문을 적용하는 것이 좋다.
- 만약 같은 타입 시그니처를 반복적으로 작성한 코드가 있다면 함수 타입을 분리해 내거나 이미 존재하는 타입을 찾아보도록 해야 한다. 라이브러리를 직접 만든다면 공통 콜백에 타입을 제공해야 한다.
- 다른 함수의 시그니처를 참조하려면 typeof fn을 사용하면 된다.

---

JS와 TS에서 함수 문장(statement)과 함수 표현식(expression)을 인식하는 방법이 다르다.

```jsx
function rollDice1(sides: number): number {/* ... */} // 문장
const rollDice2 = function(sides: number): number {/* ... */} // 표현식
const rollDice3 = (sides: number): number => {/* ... */} // 표현식
```

**타입스크립트에서는 되도록 함수 표현식을 사용하는 것이 좋다. 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언해, 함수 표현식에 이를 재사용 할 수 있다는 장점이 있다;.**

```jsx
type DiceRollFn = (sides: number) => number;
const rollDice: DiceRollFn = sides => {/* ... */};
```

편집기에서 `sides` 에 마우스를 올려 보면, 타입스크립트에서는 이미 `sides` 의 타입을 `number` 로 인식하고 있다는 것을 알 수 있다. 예시가 간단해서 함수 타입을 선언할 수 있다는 것이 장점으로 와닿지 않을 수 있으니, 이에 대한 장점을 더 알아보자.

함수 타입의 선언은 불필요한 코드의 반복을 줄인다. 사칙연산을 하는 함수 네 개는 다음과 같이 작성할 수 있다.

```jsx
function add(a: number, b: number) { return a + b };
function sub(a: number, b: number) { return a - b };
function mul(a: number, b: number) { return a * b };
function div(a: number, b: number) { return a / b };

```

반복되는 함수 시그니처를 하나의 함수 타입으로 통합할 수 있다.

```jsx
type BinaryFn = (a: number, b: number) => number;
const add: BinaryFn = (a, b) => a + b;
const sub: BinaryFn = (a, b) => a - b;
const mul: BinaryFn = (a, b) => a * b;
const div: BinaryFn = (a, b) => a / b;
```

이 예제는 함수 타입 선언을 이용했던 예제보다 타입 구문이 적다. 함수 구현부도 분리되어 있어 로직이 보다 분명해지는 장점이 있다. 모든 함수 표현식의 반환 타입까지 `number` 로 선언한 셈이다.

라이브러리는 공통 함수 시그니처를 타입으로 제공하기도 한다. 예를 들어, **리액트는 함수의 매개변수에 명시하는 `MouseEvent` 타입 대신에 함수 전체에 적용할 수 있는 `MouseEventHandler` 타입을 제공한다.** 만약 라이브러리를 직접 만들고 있다면, 공통 콜백 함수를 위한 타입 선언을 제공하는 것이 좋다.

시그니처가 일치하는 다른 함수가 있을 때도 함수 표현식에 타입을 적용해 볼만하다. 예를 들어, 웹브라우저에서 `fetch` 함수는 특정 리소스에 `HTTP` 요청을 보낸다.

```jsx
const responseP = fetch('/quote?by=Mark+Twain');
// 타입이 Promise<Response>
```

그리고 `response.json()` 또는 `response.text()` 를 사용해 응답의 데이터를 추출한다.

```jsx
async function getQuote() {
	const response = await fetch('/quote?by=Mark+Twain');
	const quote = await response.json();
	return quote;
}
// {
// "quote": "If you tell the truth, you don't have to remember anything.",
// "source": "notebook",
// "date": "1894"
// }
```

이 곳엔 버그가 존재한다. `/quote` 가 존재하지 않는 API라면, ‘404 Not Found’가 포함된 내용을 응답한다. 응답은 `JSON` 형식이 아닐 수 있다. `response.json()` 은 JSON 형식이 아니라는 새로운 오류 메시지를 담아 거절된(rejected) 프로미스를 반환한다.

호출한 곳에서는 새로운 오류 메시지가 전달되어 실제 오류인 404가 감추어진다.

또한 `fetch` 가 실패하면 거절된 프로미스를 응답하지는 않는다는 걸 간과하기 쉽다. 그러니 상태 체크를 수행해 줄 `checkedFetch` 함수를 작성해 작성해보자. `fetch` 의 타입 선언은 `lib.dom.d.ts` 에 있으며 다음과 같다.

```jsx
declare function fetch(
	input: RequestInfo, init?: RequestInit
): Promise<Response>;
```

`checkedFetch` 는 다음처럼 작성할 수 있다.

```jsx
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
	const response = await fetch(input, init);
	if (!response.ok) {
		throw new Error('Request failed: ' + response.status);
	}
	return response;
}
```

이 코드도 잘 동작하지만, 다음과 같이 더 간결하게 작성할 수 있다.

```jsx
const checkedFetch: typeof fetch = async (input, init) => {
	const response = await fetch(input, init);
	if (!response.ok) {
		throw new Error('Request failed' + response.status);
	}
	return response;
}
```

함수 문장을 함수 표현식으로 바꿨고 함수 전체에 타입(typeof fetch)을 적용했다. 이는 타입스크립트가 `input` 과 `init` 의 타입을 추론할 수 있게 해준다.

타입 구문은 또한 `checkedFetch` 의 반환 타입을 보장하며, `fetch` 와 동일하다. 예를 들어 `throw` 대신 `return` 을 사용했다면, 그 실수를 잡아낸다.

```jsx
const checkedFetch: typeof fetch = async (input, init) => {
	// ~ 'Promise<Response | Error>' 형식은
	// 'Promise<Response>' 형식에 할당할 수 없습니다.
	// 'Response | Error' 형식은
	// 'Response' 형식에 할당할 수 없습니다.
	const response = await fetch(input, init);
	if (!response.ok) {
		return new Error('Request failed: ' + response.status);
	}
	return response;
}
```

`checkedFetch` 를 함수 문장으로 작성한 예제에서도 `throw` 가 아니라 `return` 을 사용할 경우 오류가 발생한다. 그러나 오류는 첫 번째 예제와 달리 `checkedFetch` 구현체가 아닌, 함수를 호출한 위치에서 발생한다.

함수의 매개변수에 타입 선언을 하는 것보다 함수 표현식 전체 타입을 정의하는 것이 코드도 간결하고 안전하다. 다른 함수의 시그니처와 동일한 타입을 가지는 새 함수를 작성하거나, 동일한 타입 시그니처를 가지는 여러 개의 함수를 작성할 때는 매개변수의 타입과 반환 타입을 반복해서 작성하지 말고 함수 전체의 타입 선언을 적용해야 한다.
