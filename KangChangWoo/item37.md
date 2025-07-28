# Item 37

## item.37 공식 명칭에는 상표를 붙이기

*요약*

- 타입스크립트는 구조적 타이핑(덕 타이핑)을 사용하기 때문에, 값을 세밀하게 구분하지 못하는 경우가 있다. 값을 구분하기 위해 공식 명칭이 필요하다면 상표를 붙이는 것을 고려해야 한다.
- 상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있다.

---

구조작 타이핑의 특성 때문에 가끔 코드가 이상한 결과를 낼 수 있다.

다음 코드를 보자.

```jsx
interface Vector2D {
	x: number;
	y: number;
}

function calculateNorm(p: Vector2D) {
	return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm({x:3, y: 4}); // 정상, 결과는 5
const vec3D = {x: 3, y: 4, z: 1};
calculateNorm(vec3D); // 정상! 결과는 동일하게 5
```

이 코드는 구조적 타이핑 관점에서는 문제가 없기는 하지만, 수학적으로 따지면 2차원 벡터를 사용해야 이치에 맞다.

`calculateNorm` 함수가 3차원 벡터를 허용하지 않게 하려면 공식 명칭(nomi-nal typing)을 사용하면 된다. 공식 명칭을 사용하는 것은, 타입이 아니라 값의 관점에서 `Vector2D` 라고 말하는 것이다. 공식 명칭 개념을 타입스크립트에서 흉내 내려면 ‘상표(brand)’를 붙이면 된다. (비유를 들자면 ‘스타벅스’ 가 아니라 ‘커피’)

```jsx
interface Vector2D {
	_brand: '2d';
	x: number;
	y: number;
}

function vec2D(x: number, y: number): Vector2D {
	return {x, y, _brand: '2d};
}

function calculateNorm(p: Vector2D) {
	return Math,sqrt(p.x * p.x + p,y * p.y); // 기존과 동일
}

calculateNorm(vec2D(3, 4)); // 정상, 5를 반환합니다.
const vec3D = {x: 3, y: 4, z: 1};
calculateNorm(vec3D);
						// ~~~~ '_brand' 속성이 ... 형식에 없습니다.
```

상표(`_brand` )를 사용해서 `calculateNorm` 함수가 `Vector2D` 타입만 받는 것을 보장한다. 그러나 `vec3D` 값에 `_brand: '2d'` 라고 추가하는 것 같은 악의적인 사용을 막을 수는 없다. 다만 단순한 실수를 방지하기에는 충분하다.

상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있다. 타입 시스템이기 때문에 런타임 오버헤드를 없앨 수 있고, 추가 속성을 붙일 수 없는 `string` 이나 `number` 같은 내장 타입도 상표화 할 수 있다.

예를 들어, 절대 경로를 사용해 파일 시스템에 접근하는 함수를 가정해보자. 런타임에는 절대 경로(`’/’` )로 시작하는지 체크하기 쉽지만, 타입 시스템에서는 절대 경로를 판단하기 어렵기 때문에 상표 기법을 사용한다.

```jsx
type AbsolutePath = string & {_brand: 'abs'};

function listAbsolutePath(path: AbsolutePath) {
	// ...
}

function isAbsolutePath(path: string): path is AbsolutePath {
	return path.startWith('/');
}
```

`string` 타입이면서 `_brand` 속성을 가지는 객체를 만들 수는 없다. `AbsolutePath` 는 온전히 타입 시스템의 영역이다.

만약 `path` 값이 절대 경로와 상대 경로 둘 다 될 수 있다면, 타입을 정제해 주는 타입 가드를 사용해서 오류를 방지할 수 있다.

```jsx
function f(path: string) { 
	if (isAbsolutePath(path)) {
		listAbsolutePath(path);
	}
	listAbsolutePath(path);
									// ~~~~ 'string' 형식의 인수는 'AbsolutePath' 형식의
									// 매개변수에 할당될 수 없습니다.
}
```

`if` 체크로 타입을 정제하는 방식은 매개변수 `path` 가 절대 경로인지 또는 상대 경로인지에 따라 분기하기 때문에 분기하는 이유를 주석으로 붙이기에도 좋다. 단, 주석이 오류를 방지해 주는 것은 아니라는 점은 주의해야 한다.

반면, 로직을 분기하는 대신 오류가 발생한 곳에 `path as AbsolutePath` 를 사용해서 오류를 제거할 수도 있지만 단언문은 지양해야 한다. 단언문을 쓰지 않고 `AbsolutePath` 타입을 얻기 위한 유일한 방법은 `AbsolutePath` 타입을 매개변수로 받거나 타입이 맞는지 체크하는 것뿐이다.

상표 기법은 타입 시스템 내에서 표현할 수 없는 수많은 속성들을 모델링하는 데 사용되기도 한다. 예를 들어, 목록에서 한 요소를 찾기 위해 이진 검색을 하는 경우를 보자.

```jsx
function binarySearch<T>(xs: T[], x: T): boolean {
	let low = 0, high = xs.length - 1;
	while (high >= low) {
		const mid = low + Math.floor((high - low) / 2);
		const v = xs[mid];
		if (v === x) return true;
		[low, high] = x > v ? [mid + 1, high] : [low, mid - 1];
	}
	return false;
}
```

이진 검색은 이미 정렬된 상태를 가정하기 때문에, 목록이 이미 정렬되어 있다면 문제가 없다. 하지만 목록이 정렬되어 있지 않다면 잘못된 결과가 나온다. 타입스크립트 타입 시스템에서는 목록이 정렬되어 있다는 의도를 표현하기 어렵다. 따라서 다음 예제처럼 상표 기법을 사용해보자.

```jsx
type SortedList<T> = T[] & {_brand: 'sorted'};

function isSorted<T>(xs: T[]): xs is SortedList<T> {
	for (let i = 1; i < xs.length; i++) {
		if (xs[i] < xs[i - 1]) {
			return false;
		}
	}
	return true;
}

function binarySearch<T>(xs: SortedList<T>, x: T): boolean {
	// ...
}
```

`binarySearch` 를 호출하려면, 정렬되었다는 상표가 붙은 `SortedList` 타입의 값을 사용하거나 `isSorted` 를 호출하여 정렬되었음을 증명해야 한다. `isSorted` 에서 목록 전체를 루프 도는 것이 효율적인 방법은 아니지만 적어도 안전성은 확보할 수 있다.

앞의 예제는 타입 체커를 유용하게 사용하는 일반적인 패턴이다. 예를 들어, 객체의 메서드를 호출하는 경우 `null` 이 아닌 객체를 받거나 조건문을 사용해서 해당 객체가 `null` 이 아닌지 체크하는 코드와 동일한 형태이다.

`number` 타입에도 상표를 붙일 수 있다. 예를 들어, 단위를 붙여 보자.

```jsx
type Meters = number & {_brand: 'meters'};
type Seconds = number & {_brand: 'seconds'};

const meters = (m: number) => m as Meters;
const seconds = (s: number) => s as Seconds;

const oneKm = meters(1000); // 타입이 Meters
const oneMin = seconds(60); // 타입이 Seconds
```

`number` 타입에 상표를 붙여도 산술 연산 후에는 상표가 없어지기 때문에 실제로 사용하기에는 무리가 있다.

```jsx
const tenKm = oneKm * 10; // 타입이 number
const v = oneKm / oneMin; // 타입이 number
```

그러나 코드에 여러 단위가 혼합된 많은 수의 숫자가 들어 있는 경우, 숫자의 단위를 문서화하는 괜찮은 방법일 수 있다.
