# Item03

# Item03 코드 생성과 타입이 관계없음을 이해하기

*요약*

- 코드 생성은 타입 시스템과 무관하다. 타입스크립트  타입은 런타임 동작이나 성능에 영향을 주지 않는다.
- 타입 오류가 존재하더라도 코드 생성(컴파일)은 가능하다.
- 타입스크립트 타입은 런타임에 사용할 수 없다. 런타임에 타입을 지정하려면, 타입 정보 유지를 위한 별도의 방법이 필요하다.
    - 태그된 유니온, 속성 체크 방법
    - 클래스로 타입과 런타임 값 모두 제공하기

---

### 타입스크립트 컴파일러의 두 가지 역할

1. 최신 TS/JS를 브라우저에서 동작할 수 있도록 구버전의 JS로 트랜스파일(transpile, 번역+컴파일)
2. 코드의 타입 오류를 체크(정적 타입 시스템)
- 위의 두 가지는 서로 완벽히 독립적이다. 즉, 변환과 타입은 서로 영향을 주지 않는다.

### 타입 오류가 있는 코드도 컴파일이 가능하다.

```jsx
let x = 'hello';
x = 1234;

tsc.test.ts
test.ts:2:1 - error TS2322: '1234' 형식은 'string' 형식에 할당할 수 없습니다. 2 x = 1234;
```

```jsx
var x = 'hello';
x = 1234;
```

- 타입스크립트 오류는 C나 자바 같은 언어들의 경고(warning)과 비슷하다. 문제가 될 부분을 알려주지만, 빌드를 멈추지는 않는다.
- 코드에 타입 문제가 있을 때 “타입 체크에 문제가 있다” 가 정확한 표현이다.
    
    ```jsx
    코드에 오류가 있을 때 "컴파일에 문제가 있다"고 말하는 경우가 있다. 그러나 이는 기술적으로 틀린말인데,
    엄밀히 말하면 오직 코드 생성만이 '컴파일'이라고 할 수 있기 때문이다. 작성한 타입스크립트가 유효한
    자바스크립트라면 타입스크립트 컴파일러는 컴파일을 해 낸다. 그러므로 코드에 오류가 있을 때 "타입 체크에
    문제가 있다"고 말하는 것이 더 정확한 표현이다.
    ```
    
- 때로 이런 오류는 문제가 되지만, 이를 수정하지 않아도 애플리케이션의 다른 부분은 정상적으로 실행된다.
- 만약 오류가 있을 때 컴파일하지 않으려면, `tsconfig.json` 에 `noEmitOnError` 를 설정하거나 빌드 도구에 동일하게 적용하면 된다.

### 런타임에는 타입 체크가 불가능하다.

```jsx
interface Square {
	width: number;
}
interface Rectangle extends Square {
	height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
	if (shape instanceof Rectangle) {
		// ~~ 'Rectangle'은(는) 형식만 참조하지만, 여기서는 값으로 사용되고 있습니다.
		return shape.width * shape.height;
		// ~~ 'Shape' 형식에 'height' 속성이 없습니다.
	} else {
		return shape.width * shape.width;
	}
}
```

- `instanceof` 체크는 런타임에 일어나지만, Rectangle은 타입이기 때문에 런타임 시점에 아무런 역할을 할 수 없다.
- 타입스크립트의 타입은 ‘제거 가능(erasable)’하며, 실제로 JS로 컴파일 되는 과정에서 모든 인터페이스, 타입, 타입 구문은 그냥 제거된다.
- 위의 문제를 해결하는 방법으로 `height` 속성이 존재하는지 체크해볼 수 있다.

```jsx
function calculateArea(shape: Shape) {
	if ('height' in shape) {
		shape; // 타입이 Rectangle
		return shape.width * shape.height;
	} else {
		shape; // 타입이 Square
		return shape.width * shape.width;
	}
}
```

- 속성 체크는 런타임에 접근 가능한 값에만 관련되지만, 타입 체크 역시도 `shape` 의 타입을 `Rectangle` 로 보정해주기 때문에 오류가 사라진다.

- 타입 정보를 유지하는 또 다른 방법으로는 런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 ‘태그’ 기법이 있다.

```jsx
interface Square {
	kind: 'square';
	width: number;
}
interface Rectangle {
	kind: 'rectangle';
	height: number;
	width: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
	if (shape.kind === 'rectangle') {
		shape; // 타입이 Rectangle
		return shape.width * shape.height;
	} else {
		shape; // 타입이 Square
		return shape.width * shape.width;
	}
}
```

- 여기서 `Shape` 타입은 ‘태그된 유니온(tagged union)’의 한 예이다. 이 기법은 런타임에 타입 정보를 손쉽게 유지할 수 있기 때문에, 타입스크립트에서 흔하게 볼 수 있다.

- 타입(런타임 접근 불가)과 값(런타임 접근 가능)을 둘 다 사용하는 기법도 있다. 타입을 클래스로 만들면 된다. `Square` 와 `Rectangle` 을 클래스로 만들면 오류를 해결할 수 있다.

```jsx
class Square {
	constructor(public width: number) {}
}
class Rectangle extends Square {
	constructor(public width: number, public height: number) {
		super(width);
	}
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
	if (shape instanceof Rectangle) {
		shape; // 타입이 Rectangle
		return shape.width * shape.height;
	} else {
		shape; // 타입이 Square
		return shape.width * shape.width; // 정상
	}
}
```

- 인터페이스는 타입으로만 사용 가능하지만, `Rectangle` 을 클래스로 선언하면 타입과 값으로 모두 사용할 수 있으므로 오류가 없다.
- `type Shape = Square | Rectangle`  부분에서 `Rectangle` 은 타입으로 참조되지만, `shape instanceof Rectangle` 부분에서는 값으로 참조된다.

### 타입 연산은 런타임에 영향을 주지 않는다.

- `string`  또는 `number` 타입인 값을 항상 `number` 로 정제하는 경우를 가정해보자. 아래 코드는 타입 체커를 통과하지만 잘못된 방법을 사용한다.
    
    ```jsx
    function asNumber(val: number | string): number {
    	return val as number;
    }
    ```
    
    - 변환된 자바스크립트 코드는 아래와 같다.
    
    ```jsx
    function asNumber(val) {
    	return val;
    }
    ```
    
    - 코드에 어떤 정제 과정도 존재하지 않는다. `as number` 는 타입 연산이고 런타임 동작에는 아무런 영향을 미치지 않는다. 값을 정제하기 위해서는 런타임의 타입을 체크해야 하고 자바스크립트 연산을 통해 변환을 수행해야 한다.
    
    ```jsx
    function asNumber(val: number | string): number {
    	return typeof(val) === 'string' ? Number(val) : val;
    }
    ```
    

### 런타임 타입은 선언된 타입과 다를 수 있다.

```jsx
function setLightSwitch(value: boolean) {
	switch (value) {
		case true:
			turnLightOn();
			break;
		case false:
			turnLightOff();
			break;
		default:
			console.log(`실행되지 않을까 봐 걱정됩니다.`);
	}
}
```

- 타입스크립트는 일반적으로 실행되지 못하는 죽은(dead) 코드를 찾아내지만, 여기서는 `strict` 를 설정하더라도 찾아내지 못한다.
- 마지막 부분이 `:boolean` 이라는 타입 선언문이며, 런타임 시 제거된다. 자바스크립트라면 `setLightSwitch("ON")` 으로 호출하면 default 문에 닿을 수 있다.
- 타입스크립트에서는 런타임 타입과 선언된 타입이 맞지 않을 수 있다. 타입이 달라지는 혼란스러운 상황을 피해야하며, 언제든지 선언된 타입이 달라질 수 있다는 것을 명심해야 한다.

### 타입스크립트 타입으로는 함수를 오버로드할 수 없다.

```jsx
function add(a: number, b: number) { return a+b; }
	// ~~ 중복된 함수 구현입니다.
function add(a: string, b: string) { return a+b; }
	// ~~ 중복된 함수 구현입니다.
```

- 타입스크립트에서는 타입과 런타임의 동작이 무관하기에, 함수 오버로딩은 불가능하다.
- TS가 함수 오버로딩을 지원하기는 하나, 타입 수준에서만 동작할 뿐이다. 하나의 함수에 대해 여러 개의 선언문을 작성할 수 있지만, 구현체(implementtation)는 오직 하나뿐이다.
    
    ```jsx
    function add(a, b) {
    	return a + b;
    }
    
    const three = add(1, 2); // 타입이 number
    const twelve = add('1', 2); // 타입이 string
    ```
    
- `add` 에 대한 처음 두 개의 선언문은 타입 정보를 제공할 뿐이다. 이 두 선언문은 타입스크립트가 자바스크립트로 변환되면서 제거되며, 구현체만 남게 된다.
    - 오버로드 시그니쳐
        
        자바스크립트에선 함수 이름을 중복해 선언할 수 없다. 대신, 타입은 여러개 선언할 수 있는데, 이걸 오버로드 시그니쳐 라고 부른다. 따라서 타입스크립트 컴파일 시 사라지게 된다.
        

### 타입스크립트 타입은 런타임 성능에 영향을 주지 않는다.

- 타입과 타입 연산자는 자바스크립트 변환 시점에 제거되기 때문에, 런타임의 성능에 아무런 영향을 주지 않는다.
- 타입스크립트의 정적 타입은 실제로 비용이 전혀 들지 않는다.
- ‘런타임’ 오버헤드가 없는 대신, 타입스크립트 컴파일러는 ‘빌드타임’ 오버헤드가 있다. 따라서 컴파일은 일반적으로 빠르며, 특히 증분(incre-mental) 빌드 시 더욱 체감된다.
- 오버헤드가 커지면 빌드 도구에서 ‘transpile only’ 옵션을 설정해 타입 체크를 건너뛸 수 있다.
