# Item07

## Item.07 타입이 값들의 집합이라고 생각하기

*요약*

- 타입을 값의 집합으로 생각하면 이해하기 편하다. (타입의 ‘범위’) 이는 유한하거나 무한하다.
- 타입스크립트 타입은 엄격한 상속 관계가 아니라 겹쳐지는 집합(벤 다이어그램)으로 표현된다. 두 타입은 서브 타입이 아니면서도 겹쳐질 수 있다.
- 한 객체의 추가적인 속성이 타입 선언에 언급되지 않더라도, 그 타입에 속할 수 있다.
- 타입 연산은 집합의 범위에 적용된다. A와 B의 인터섹션은 A의 범위와 B의 범위의 인터섹션이다. 객체 타입에서는  A & B인 값이 A와 B의 속성을 모두 가짐을 의미한다.
- ‘A는 B를 상속’, ‘A는 B에 할당 가능’, ‘A는 B의 서브 타입’ 은 ‘A는 B의 부분 집합’ 과 같은 의미를 갖는다.

---

### 타입은 ‘할당 가능한 값들의 집합’ 이다.

- 런타임에 모든 변수는 자바스크립트에서 존재하는 다양한 고유 값을 가진다.
- 코드가 실행되기 전에는 타입을 갖게 된다. 이 타입은 일종의 ‘범위’ 또는 ‘집합’ 과 같은 개념으로 모든 숫자 값은 `number` 타입에 속하지만 문자는 아닌 것이 그 예이다.
- `null` 과 `undefined` 는 `strictNullChecks` 여부에 따라 `number` 에 해당될 수도, 아닐 수도 있다.
- 가장 작은 집합은 공집합으로, TS에선 `never` 타입이다. `never` 타입으로 선언된 변수의 범위는 공집합이므로 아무 값도 할당할 수 없다.
    
    ```jsx
    const x: nver = 12;
    // ~ '12' 형식은 'never' 형식에 할당할 수 없습니다.
    ```
    

그 다음으로 작은 집합은 한 가지 값만 포함하는 타입이다. 이는 **unit 타입**이라고도 불리는 **리터럴 타입**이다.

```jsx
type A = 'A'
type B = 'B'
type Twelve = '12';
```

이를 두 개 혹은 세 개로 묶기 위해 유니온 타입을 사용한다. 유니온 타입은 값 집합들의 합집합을 일컫는다.

```jsx
type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12;
```

### 타입 체커의 주요 역할은 부분 집합 검증이다.

```jsx
const a: AB = 'A'; // 정상, 'A'는 집합 {'A', 'B'} 의 원소입니다.
const c: AB = 'C'; // ~ '"C"' 형식은 'AB' 형식에 할당할 수 없습니다.
```

“C” 는 유닛 타입이다. 범위는 단일 값 “C”로 구성되며, AB(”A”와 “B”로 이루어진)의 부분 집합이 아니므로 오류이다.

```jsx
// 정상, {"A", "B"}는 {"A", "B", 12}의 부분 집합입니다.
type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12;

const ab: AB = Math.random() < 0.5 ? 'A' : 'B';
const ab12: AB12 = ab; // OK

declare let twelve: AB12;
const back: AB = twelve;
// 'AB12' 형식은 'AB' 형식에 할당할 수 없습니다.
// '12' 형식은 'AB' 형식에 할당할 수 없습니다.
```

`back` 은 AB 타입이므로 `12` 를 허용하지 않는다. 즉, 부분 집합에 포함되지 않는 타입이 있어서 타입 체커가 오류를 반한한다.

```jsx
type Int = 1 | 2 | 3 | 4 | 5; // ...
```

실무에서 실제로 다루는 타입은 대부분 범위가 무한대이다. 이는 개별 타입을 일일이 추가해서 만든 타입으로 생각할 수 있다.

```jsx
interface Identified {
	id: string;
}
// Identified 속성은 반드시 string 타입의 id를 가져야 한다!
```

어떤 객체가 `string` 으로 할당 가능한 `id` 속성을 가지고 있다면, 그 객체는 `Identified` 이다.

구조적 타이핑 규칙은 어떤 값이 다른 속성도 가질 수 있음을 의미한다.

함수 호출의 매개변수 역시 다른 속성을 가진다.

- 타입의 구조적 타이핑
    
    ```jsx
    const user = {
      id: "abc123",
      name: "Alice",
      age: 30,
    };
    
    function printId(item: Identified) {
      console.log(item.id);
    }
    
    printId(user); // 정상
    ```
    
    이 예시가 바로 구조적 타이핑과 타입에 대한 내용이다. `user` 는 `id` 를 갖는다.
    
    `Identified` 는 `string` 타입의 `id` 만 있으면 되므로, `name` 이나 `age` 가 있어도 문제가 되지 않는다.
    
    구조적 타이핑이란 타입의 이름이나 속성이 아니라, **구조**만 보는 것이다.
    
    즉,’ 객체가 이름은 달라도 속성만 같으면(가지고 있으면) 똑같다’ 라고 보는 것이다.
    
    ```jsx
    const thing = {
      name: "No ID",
    };
    
    printId(thing); // id 속성이 없음
    ```
    
    **함수 호출의 매개변수 역시 다른 속성을 가진다?**
    
    ```jsx
    function printId(item: Identified) {
      console.log(item.id);
    }
    
    printId({ id: "xyz", extra: "value" }); // 정상
    ```
    
    `id` 속성만 있으면 같은 객체(타입)으로 판단해, 정상 작동한다.
    
- 잉여 속성 검사
    
    ```jsx
    interface Person {
      name: string;
    }
    
    const p = {
      name: "Alice",
      age: 25, // 타입에는 없지만 OK
    };
    
    function greet(person: Person) {
      console.log("Hello, " + person.name);
    }
    
    greet(p); // 정상
    ```
    
    `p` 객체에는 `age` 라는 속성도 있지만, 변수로 전달할 경우 타입 체커는 “다른 곳에서도 사용한다.” 라고 판단하고 오류를 뱉지 않는다.
    
    ```jsx
    greet({
      name: "Bob",
      age: 30, // X. Person에 없는 속성
    });
    ```
    
    하지만 리터럴로 직접 넘기게 되면, 정의되지 않는 속성을 사용한다고 판단해 오류를 낸다. 이것이 바로 잉여 속성 검사이다.
    
    즉, 잉여 속성이란 “지정된 속성이 아닌, 다른(남는) 속성”을 말하는 것이다.
    
    이를 우회하기 위해 두 가지 방법을 사용할 수 있는데,
    
    1. 변수에 담아 우회
    2. `as` 타입단언 사용
        
        ```jsx
        greet({
          name: "Bob",
          age: 30, // X. Person에 없는 속성
        } as Person);
        ```
        
        **타입 단언은 컴파일러(tsc)가 타입을 강제로 신뢰하게 만들기 때문에, 타입 검사를 우회하게 되고, 결과적으로 타입 안정성이 보장되지 않아 위험할 수 있다.**
        

```jsx
type K = keyof (Person | Lifespan); // 타입이 never

type Person = { name: string; age: number; }
type Lifespan = { birthDate: Date; deathDate: Date; }

type K = keyof (Person | Lifespan); // never

keyof (A&B) = (keyof A) | (keyof B)
keyof (A|B) = (keyof A) & (keyof B)
```

위 예시에서 `Person` 과 `Lifespan` 의 공통 속성이 없으므로 `never` 타입을 갖는다.

### `extends`  키워드

```jsx
interface Person {
	name: string;
}

interface PersonSpan extends Person {
	birth: Date;
	death?: Date;
}
```

타입은 집합이라고 볼 수 있기에 `extends` 의 의미는 ‘~에 할당 가능한’, ‘~의 부분 집합’ 이라는 의미로 받아들일 수 있다.

`PersonSpan` 타입의 모든 값은 문자열 `name` 속성을 가져야 한다. 그리고 `birth` 속성을 가져야 제대로 된 부분집합이 된다.

```jsx
interface Vector1D { x: number; }
interface Vector2D extends Vector1D { y: number; }
interface Vector3D extends Vector2D { z: number; }

interface Vector1D { x: number; }
interface Vector2D { x: number; y: number;}
interface Vector3D { x: number; y: number; z: number;}
```

Vector3D는 Vector2D의 서브 타입이고, Vector2D는 Vector1D의 서브 타입이다. 클래스 관점에서는 서브 클래스로 볼 수 있다.

```jsx
function getKey <K extends string> (val: any, key: K) { ... }
```

`string` 을 상속한다는 의미를 집합의 관점으로 생각하자. `string` 의 부분 집합 범위를 가지는 어떠한 타입이 된다. 이 타입은 `string` 리터럴 타입, `string` 리터럴 타입의 유니온, `string` 자신을 포함한다.

```jsx
getKey({}, 'x'), // 정상, 'x'는 string을 상속
getKey({}, Math.random() < 0.5 : 'a' : 'b'); // 정상, 'a' | 'b' 는 string을 상속
getKey({}, document.title); // 정상, 'a' | 'b'는 string을 상속
getKey({}, 12); // ~ '12' 형식의 인수는 'string' 형식의 매개변수에 할당될 수 없습니다.
```

마지막 오류는 상속의 관점에서 “상속할 수 없다”로 받아들일 수 있고, ‘~의 부분 집합’의 의미로 받아들일 수 있다.
