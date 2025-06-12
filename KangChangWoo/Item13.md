# Item 13

## Item.13 타입과 인터페이스의 차이점 알기

요약

- 타입과 인터페이스의 차이점과 비슷한 점을 이해해야 한다.
- 한 타입을 `type` 과 `interface` 두 가지 문법을 사용해서 작성하는 방법을 터득해야 한다.
- 프로젝트에서 어떤 문법을 사용할지 결정할 때 한 가지 일관된 스타일을 확립하고, 보강 기법이 필요한지 고려해야 한다.

---


타입스크립트에서 명명된 타입(named type)을 정의하는 방법은 두 가지가 있다. 아래처럼 타입을 사용할 수 있다.

```jsx
type Tstate = {
	name: string;
	capital: string;
}
```

또는 인터페이스를 사용해도 된다.

```jsx
interface Istate {
	name: string;
	capital: string;
}
```

(명명된 타입을 정의할 때 인터페이스 대신 클래스를 사용할 수도 있지만, 클래스는 값으로도 쓰일 수 있는 자바스크립트 런타임의 개념이다.)

대부분의 경우 타입을 사용해도 되고 인터페이스를 사용해도 된다. 그러나 타입과 인터페이스 사이에 존재하는 차이를 분명하게 알고, 같은 상황에서는 동일한 방법으로 명명된 타입을 정의해 일관성을 유지해야 한다. 그러려면 하나의 타입에 대해 두 가지 방법을 모두 정의할 줄 알아야 한다.

### 인터페이스 선언과 타입 선언의 비슷한 점

명명된 타입은 인터페이스로 정의하든 타입으로 정의하든 상태에는 차이가 없다. 만약 `IState` 와 `TState` 를 추가 속성과 함께 할당한다면 동일한 오류가 발생한다.

```jsx
const wyoming: TState = {
	name: 'Wyoming',
	capital: 'Cheyenne',
	population: '500_000'
	// ~~ ... 형식은 'TState' 형식에 할당할 수 없습니다.
	// 개체 리터럴은 알려진 속성만 지정할 수 있으며
	// 'TState' 형식에 'population' 이(가) 없습니다.
}
```

인덱스 시그니처는 인터페이스와 타입에서 모두 사용할 수 있다. 

```jsx
type TDict = { [key: string]: string };
interface IDict {
	[key: string]: string;
}
```

또한 함수 타입도 인터페이스나 타입으로 정의할 수 있다.

```jsx
type TFn = (x: number) => string;
interface TFn {
	(x: number) : string;
}

const toStrT: TFn = x => '' + x; // 정상
const toStrI: TFn = x => '' + x; // 정상
```

이런 단순 함수 타입에는 타입 별칭(alias)이 더 나은 선택이겠지만, 함수 타입에 추가적인 속성이 있다면 타입이나 인터페이스 어떤 것을 선택하든 차이가 없다.

```jsx
type TFnWithProperties = {
	(x: number): number;
	prop: string;
}
interface IFnWithProperties {
	(x: number): number;
	prop: string;
}
```

문법이 생소할 수도 있지만 자바스크립트에서 함수는 호출 가능한 객체라는 것을 떠올려 보면 납득할 수 있는 코드다.

타입 별칭과 인터페이스는 모두 제너릭이 가능하다.

```jsx
type TPair<T> = {
	first: T;
	second: T;
}
interface IPair<T> {
	first: T;
	second: T;
}
```

인터페이스는 타입을 확장할 수 있으며, 타입은 인터페이스를 확장할 수 있다.

```jsx
interface IStateWithPop extends TState {
	population: number;
}
type TStateWithPop = Istate & { population: number };
```

`IStateWithPop` 과 `TStateWithPop` 은 동일하다. 여기서 **주의할 점은 인터페이스는 유니온 타입 같은 복잡한 타입을 확장하지는 못한다**는 것이다. **복잡한 타입을 확장하고 싶다면 타입과 `&` 를 사용해야 한다.**

한편 클래스를 구현(Implemets)할 때는, 타입(`TState` )과 인터페이스(`IState`) 둘 다 사용할 수 있다.

```jsx
class StateT implements TState {
	name: string = '';
	capital: string = '';
}
class StateI implements IState {
	name: string = '';
	capital: string = '';
}
```

### 타입과 인터페이스의 다른 점

유니온 타입은 있지만 유니온 인터페이스는 없다.

```jsx
type AorB = 'a' | 'b';
```

인터페이스는 타입을 확장할 수 있지만, 유니온은 할 수 없다. 그런데 유니온 타입을 확장하는 게 필요할 때가 있다.

`Input` 과 `Output` 은 별도의 타입이며 이 둘을 하나의 변수명으로 매핑하는 `VariableMap` 인터페이스를 만들 수 있다.

```jsx
type Input = { /* ... */ };
type Output = { /* ... */ };
interface VariableMap {
	[name: string]: Input | Output;
}
```

또는 유니온 타입에 `name` 속성을 붙인 타입을 만들 수도 있다.

```jsx
type NamedVariable = (Inpput | Output) & { name: string };
```

이 타입은 인터페이스로 표현할 수 없다. `type` 키워드는 일반적으로 `interface` 보다 쓰임새가 많다. `type` 키워드는 유니온이 될 수도 있고, 매핑된 타입 또는 조건부 타입 같은 고급 기능에 활용되기도 한다.

튜플과 배열 타입도 `type` 키워드를 이용해 더 간결하게 표현할 수 있다.

```jsx
type Pair = [number, number];
type StringList = string[];
type NamedNums = [string, ...number[]];
```

인터페이스로도 튜플과 비슷하게 구현할 수 있기는 하다.

```jsx
interface Tuple {
	0: number;
	1: number;
	length: 2;
}
const t: Tuple = [10, 20]; // 정상
```

그러나 인터페이스로 튜플과 비슷하게 구현하면 튜플에서 사용할 수 있는 `concat` 과 같은 메서드들을 사용할 수 없다. 그러므로 튜플은 `type` 키워드로 구현하는 것이 낫다.

반면 인터페이스는 타입에 없는 몇 가지 기능이 있다. 그중 하나는 바로 ‘보강(`augment`)’ 이 가능하다는 것이다. 이번 아이템 처음에 등장했던 `State` 예제에 `population` 필드를 추가할 때 보강 기법을 사용할 수 있다.

```jsx
interface IState {
	name: string;
	capital: string;
}
interface IState {
	population: number;
}
const wyoming: IState = {
	name: 'Wyoming',
	capital: 'Cheyenne',
	population: 500_000
}; // 정상
```

이 예제처럼 속성을 확장하는 것을 ‘선언 병합(`declaration merging`)’이라고 한다.

선언 병합은 주로 타입 선언 파일에서 사용된다. 따라서 타입 선언 파일을 작성할 때는 선언 병합을 지원하기 위해 반드시 인터페이스를 사용해야 하며 표준을 따라야 한다. 타입 선언에는 사용자가 채워야 하는 빈틈이 있을 수 있는데, 바로 이 선언 병합이 그렇다.

타입스크립트는 여러 버전의 자바스크립트 표준 라이브러리에서 여러 타입을 모아 병합한다. 예를 들어, `Array` 인터페이스는 `lib.es5.d.ts` 에 정의되어 있고 기본적으로는 `lib.es5.d.ts` 에 선언된 인터페이스가 사용된다.

그러나 `tsconfig.json` 의 `lib` 목록에 `ES2015` 를 추가하면 타입스크립트는 `lib.es2015.d.ts` 에 선언된 인터페이스를 병합한다. 여기에는 ES2015에 추가된 또 다른 `Array` 선언의 `find` 같은 메서드가 포함된다. 이들은 병합을 통해 다른 `Array` 인터페이스에 추가된다. 결과적으로 각 선언이 병합되어 전체 메서드를 가지는 하나의 `Array` 타입을 얻게 된다.

병합은 선언처럼 일반적인 코드라서 언제든지 가능하다는 것을 알아야한다. 그러므로 **프로퍼티가 추가되는 것을 원하지 않는다면 인터페이스 대신 타입을 사용해야 한다.**

타입과 인터페이스 중 어느 것을 사용해야 할지 결론을 내려보자.

복잡한 타입이라면 고민할 것도 없이 타입 별칭을 사용하면 된다. 그러나 타입과 인터페이스, 두 가지 방법으로 모두 표현할 수 있는 간단한 객체 타입이라면 일관성과 보강의 관점에서 고려해 봐야 한다. 일관되게 인터페이스를 사용하는 코드베이스에서 작업하고 있다면 인터페이스를 사용하고, 일관되게 타입을 사용 중이라면 타입을 사용하면 된다.

아직 스타일이 확립되지 않은 프로젝트라면, 향후에 보강의 가능성이 있을지 생각해봐야 한다. 어떤 API에 대한 타입 선언을 작성해야 한다면 인터페이스를 통해 새로운 필드를 병합할 수 있어 유용하기 때문이다.


### 추가
- class와 implements
    
    ```jsx
    class StateT implements TState {
    	name: string = '';
    	capital: string = '';
    }
    class StateI implements IState {
    	name: string = '';
    	capital: string = '';
    }
    
    type, interface, implements => TS
    class => JS (런타임 존재)
    
    class MyClass implements MyInterface {} 가 어떻게 처리되는가?
    => implements는 TS에만 존재하기 때문에 실제로 트랜스파일된 JS 코드에선
    	 클래스만 존재한다.
    	 
    !implements 뒤에 올 수 없는 타입도 있다.
    => 유니언 타입과 함수 타입은 클래스가 구현할 수 없어 불가능하다.
    
    type Weird = { name: string } | { capital: string };
    
    class C implements Weird {} // 오류: 유니언 타입은 구현 불가
    ```
    
- 선언 병합(보강)이란?
    
    ```jsx
    interface IState {
      name: string;
      capital: string;
    }
    
    interface IState {
      population: number;
    }
    
    => 인터페이스의 이름이 같다면 자동으로 합쳐진다!
    
    interface IState {
      name: string;
      capital: string;
      population: number;
    }
    
    그러나 타입은 안된다!
    
    ** 그럼 선언 병합을 언제 사용하나? **
    => 타입 선언 파일(d.ts)에서 외부 라이브러리를 확장할 경우 많이 사용
    
    // some-library.d.ts (어떤 라이브러리에 정의된 타입)
    interface Window {
      customProperty: string;
    }
    
    // 내가 window 객체를 확장하고 싶다면?
    interface Window {
      myAppData: number;
    }
    
    => Window 타입에 customProperty와 myAppData 둘 다 존재하도록
    	자동으로 병합됨.
    ```
    
- 언제 type을 쓰고 언제 interface를 쓰는가?
    
    ```jsx
    기준
    - 일관성
    - 보강(선언 병합)의 필요 여부
    - 복잡도
    
    * 복잡한 타입이면 무조건 type
    * 라이브러리 타입 확장해야 한다면 interface
    * 내부 로직에서 쓰는 단순 객체 타입이라면?
    	- 팀의 스타일을 일관적으로 유지
    	- 새 프로젝트라면 대체로 type을 추천
    	
    * 왜 새 프로젝트는 type을 추천하나?
    - 인터페이스는 자동 병합(선언 병합)이 되기 때문
    
    interface User {
      name: string;
    }
    interface User {
      age: number; // 누가 어디선가 병합했는데 몰랐다면?
    }
    - 내부에서만 쓰는 타입에는 interface를 피하자는 뜻
    (외부 라이브러리나 다른 모듈과 연결되지 않은,
    	우리 프로젝트 안에서만 사용하는 타입)
    ```
    

그러나 프로젝트 내부적으로 사용되는 타입에 선언 병합이 발생하는 것은 잘못된 설계이다. 따라서 이럴 때는 타입을 사용해야 한다.
