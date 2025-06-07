# Item 11

## Item.11 잉여 속성 체크의 한계 인지하기

*요약*

- **객체 리터럴을 변수에 할당**하거나 **함수에 매개변수로 전달할 때** 잉여 속성 체크가 수행된다.
- 잉여 속성 체크는 오류를 찾는 효과적인 방법이지만, 타입스크립트 타입 체커가 수행하는 일반적인 구조적 할당 가능성 체크와 역할이 다르다. 할당의 개념을 정확히 알아야 **잉여 속성 체크**와 **일반적인 구조적 할당 가능성 체크**를 구분할 수 있다.
- 잉여 속성 체크에는 한계가 있다. **임시 변수**를 도입하면 잉여 속성 체크를 건너뛸 수 있다는 점을 기억해야 한다.

---

타입이 명시된 변수에 객체 리터럴을 할당할 때, 타입스크립트는 **해당 타입의 속성이 있는지**, 그리고 **‘그 외의 속성은 없는지’** 확인한다.

```jsx
interface Room {
	numDoors: number;
	ceilingHeightFt: number;
}

const r: Room = {
	numDoors: 1,
	ceilingHeightFt: 10,
	elephant: 'present',
	// ~ 개체 리터럴은 알려진 속성만 지정할 수 있으며
	//   'Room' 형식에 'elephant' 이(가) 없습니다.
}

=> 해당 타입에 명시되지 않은 속성을 포함 >> [잉여 속성 체크]

const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present' // Room에는 없는 속성
};

const r: Room = obj; // OK — 구조는 만족, 잉여 속성 검사 안 함

=> elephant는 Room에 없는 속성이지만
변수에 담는 순간 '넓은 객체'로 인식함. (본문 : Room 타입의 부분 집합을 포함)
=> 잉여 속성 검사를 진행하지 않음. 타입체커도 통과. Room에 할당 가능
```

`Room` 타입에 생뚱맞게 `elephant` 속성이 있는 것이 어색하긴 하지만, 구조적 타이핑 관점으로 생각해 보면 오류가 발생하지 않아야 한다. 임시 변수를 토입해보면 알 수 있는데, `obj` 객체는 `Room` 타입에 할당 가능하다.

`obj` 의 타입은 `{ numDoors: number; ceilingHeightFt: number; elephant: string; }` 으로 추론된다. `obj` 타입은 `Room` 타입의 부분 집합을 포함하므로, `Room` 에 할당 가능하며 타입 체커도 통과한다.

두 예제의 차이점을 살펴보자. 첫 번째 예제에서, 구조적 타입 시스템에서 발생할 수 있는 중요한 종류의 오류를 잡을 수 있도록 ‘잉여 속성 체크’ 라는 과정이 수행됐다.

그러나, 잉여 속성 체크 역시 조건에 따라 동작하지 않는다는 한계가 있고, 통상적인 할당가능 검사와 함께 쓰이면 구조적 타이핑이 무엇인지 혼란스러워질 수 있다. 잉여 속성 체크가 할당 가능 검사와는 별도의 과정이라는 것을 알아야 타입스크립트 타입 시스템에 대한 개념을 잡을 수 있다.

타입스크립트는 단순히 런타임에 예외를 던지는 코드에 오류를 표시하는 것뿐 아니라, 의도와 다르게 작성된 코드까지 찾으려고 한다.

```jsx
interface Options {
	title: string;
	darkMode?: boolean;
}

function createWindow(options: Options) {
	if (options.darkMode) {
		setDarkMode();
	}
	// ...
}
createWindow({
	title: 'Spider Solitaire',
	darkmode: true
	// ~ 개체 리터럴은 알려진 속성만 지정할 수 있지만
	// 'Options' 형식에 'darkmode'이(가) 없습니다.
	// 'darkMode' 을(를) 쓰려고 했습니까?
});
```

해당 코드를 실행하면 런타임에 어떠한 종류의 오류도 발생하지 않지만, 타입스크립트가 알려주는 오류 메시지처럼 의도한 대로 동작하지 않을 수 있다. 오류가 발생한 부분은 `darkmode` 가 아닌 `darkMode` (대문자 M)이어야 한다.

`Options`  타입은 범위가 매우 넓기 때문에, 순수한 구조적 타입 체커는 이런 종류의 오류를 찾아내지 못한다. `darkMode`  속성에 `boolean` 타입이 아닌 다른 타입의 값이 지정된 경우를 제외하면, `string` 타입인 `title` 속성과 ‘또 다른 어떤 속성’을 가지는 모든 객체는 `Options` 타입의 범위에 속한다.

타입 스크립트 타입은 범위가 아주 넓어질 수 있다.

```jsx
const 01: Options = document; // 정상
const 02: Options = new HTMLAnchorElement; // 정상
```

`document` 와 `HTMLAnchorElement` 의 인스턴스 모두 `string` 타입인 `title` 속성을 가지고 있기 때문에 할당문은 정상이다. `Options` 은 정말 넓은 타입임을 알 수 있다.

잉여 속성 체크를 이용하면 기본적으로 타입 시스템의 구조적 본질을 해치지 않으면서도 객체 리터럴에 알 수 없는 속성을 허용하지 않음으로써, 앞에서 다룬 `Room` 이나 `Options` 예제 같은 문제점을 방지할 수 있다. (그래서 ‘엄격한 객체 리터럴 체크’ 라고도 불린다.)

`document` 나 `new HTMLAnchorElement` 는 **객체 리터럴이 아니기 때문에** 잉여 속성 체크가 되지 않는다. 그러나 `{title, darkmode}` 객체는 체크가 된다.

```jsx
const o: Options = {darkmode: true, title: 'Ski Free'};
	// ~ 'Options' 형식에 'darkmode' 이(가) 없습니다.
```

오류가 사라지는 이유를 알아내기 위해 타입 구문 없는 임시 변수를 사용해보자.

```jsx
const intermediate = {darkmode: true, title: 'Ski Free'};
const o: Options = intermediate; // 정상
```

첫 번째 줄의 오른쪽은 객체 리터럴이지만, 두 번째 줄의 오른쪽(`intermediate` )은 객체 리터럴이 아니다. 따라서 잉여 속성 체크가 적용되지 않고 오류는 사라진다.

잉여 속성 체크는 타입 단언문을 사용할 때도 적용되지 않는다.

```jsx
const o = {darkmode: true, title: 'Ski Free'} as Options;
// 정상
```

이 예제가 단언문보다 선언문을 사용해야 하는 단적인 이유 중 하나이다.

잉여 속성 체크를 원하지 않는다면, 인덱스 시그니처를 사용해서 타입스크립트가 추가적인 속성을 예상하도록 할 수 있다.

```jsx
interface Options {
	darkMode?: boolean;
	[otherOptions: string]: unknown;
}

const o: Options = {darkmode: true}; // 정상
```

선택적 속성만 가지는 ‘약한’(weak) 타입에도 비슷한 체크가 동작한다.

```jsx
interface LineChartOptions {
	logscale?: boolean;
	invertedYAxis?: boolean;
	areaChart?: boolean;
}

const opts = {logScale: true};
const o: LineChartOptions = opts;
	// ~ '{logScale: boolean;}' 유형에
	// 'LineChartOptions' 유형과 공통적인 속성이 없습니다.
```

구조적 관점에서 `LineChartOptions` 타입은 모든 속성이 선택적이므로 모든 객체를 포함할 수 있다. 이런 약한 타입에 대해서 타입스크립트는 값 타입과 선언 타입에 공통된 속성이 있는지 확인하는 별도의 체크를 수행한다.

공통 속성 체크는 잉여 속성 체크와 마찬가지로 오타를 잡는 데 효과적이며, 구조적으로 엄격하지 않다. 그러나 잉여 속성 체크와 다르게, 약한 타입과 관련된 할당문마다 수행된다. 임시 변수를 제거하더라도 공통 속성 체크는 여전히 동작한다.

잉여 속성 체크는 구조적 타이핑 시스템에서 허용되는 속성 이름의 오타 같은 실수를 잡는 데 효과적인 방법이다. 선택적 필드를 포함하는 `Options`  같은 타입에 특히 유용한 반면, 적용 범위도 매우 제한적이며 오직 객체 리터럴에만 적용된다.

이러한 한계점을 인지하고 잉여 속성 체크와 일반적인 타입 체크를 구분한다면, 두 가지 모두의 개념을 잡는 데에 도움이 될 것이다.
