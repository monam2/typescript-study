# Item08

## Item.08 타입 공간과 값 공간의 심벌 구분하기

*요약*

- 타입스크립트 코드를 읽을 때 타입인지 값인지 구분하는 방법을 터득해야 한다.
- 모든 값은 타입을 가지지만, 타입은 값을 가지지 않는다. `type` 과 `interface` 같은 키워드는 타입 공간에만 존재한다.
- `class` 나 `enum` 같은 키워드는 타입과 값 두 가지로 사용될 수 있다.
- `"foo"` 는 문자열 리터럴이거나, 문자열 리터럴 타입일 수 있다. 차이점을 알고 구별하는 방법을 터득해야 한다.
- `typeof` , `this` 그리고 많은 다른 연산자들과 키워드들은 타입 공간과 값 공간에서 다른 목적으로 사용될 수 있다.

---

*Item08에서 말하는 심벌(symbol)은 ES6의 `Symbol()` 과는 다른, 그저 값을 구분하는 식별자로써 말하는 것으로 보인다.*

### 어떤 식별자가 타입인지 값인지는 문맥으로 구분한다.

```jsx
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height });

function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape.radius;
    // ~ '{}' 형식에 'radius' 속성이 없습니다.
  }
}
```

위 코드에서 `calculateVolume` 은 `shape` 의 타입을 체크하려는 의도를 갖고 있다.

그러나, `instanceof` 는 런타임에 동작하는 연산자이기에, 값에 대한 연산을 진행하게 되어 결과적으로 이 코드에서 `Cylinder` 는  `interface Cylinder` 가 아닌, `const Cylinder` 가 된다.

```jsx
type T1 = "string literal";
type T2 = 123;
const v1 = "string literal";
const v2 = 123;
```

일반적으로 `type` 이나 `interface` 다음에 나오는 심벌은 타입인 반면, `const` 나 `let` 선언에 쓰이는 것은 값이다. 이를 쉽게 구분하려면 타입스크립트 컴파일러를 이용해 컴파일하면 타입 정보는 제거되기 때문에 값과 타입을 확실히 구분할 수 있다.

### 값과 타입은 번갈아 나올 수 있다.

```jsx
interface Person {
  first: string;
  last: string;
}

const p: Person = { first: "Jane", last: "Jacobs" };
// p: 값 / Person : 타입
```

타입스크립트 코드에서 타입과 값은 번갈아 나올 수 있다.

타입 선언 ( : ) 또는 단언문 `as`  다음 나오는 심벌은 타입인 반면, `=` 다음에 나오는 모든 것들은 값이다.

일부 함수에서는 타입과 값이 반복적으로 번갈아 가며 나올 수도 있다.

```jsx
function email(p: Person, subject: string, body: string): Response {}
```

### class와 enum은 상황에 따라 타입과 값 두 가지 모두 가능하다.

```jsx
class Cylinder {
  radius = 1;
  height = 1;
}

function calculateVolume(shape: unknown) {
  if (shape instanceof Cylinder) {
    shape; // 정상, 타입은 Cylinder
    shape.radius; // 정상, 타입은 number
  }
}
```

`class` 는 앞서 살펴봤듯이 `type` 으로도 사용 가능하기 때문에 `instanceof` 로 분류가 가능하다.

클래스가 타입으로 사용되는 경우엔 형태(속성과 메서드)가 사용되는 반면, 값으로 쓰일 때는 생성자가 사용된다.

- 클래스를 타입으로 사용하게 되는 과정
    
    ```jsx
    // 여기서 Cylinder는 생성자 함수
    const myCylinder = new Cylinder();
    
    // instanceof에서도 값으로 사용됨
    if (shape instanceof Cylinder) {
      // 여기서 Cylinder는 생성자 함수를 가리킴
    }
    ```
    
    `instanceof` 는 런타임에 “이 객체가 생성자로 만들어졌는가?” 를 체크하기 때문에 `Cylinder` 는 생성자로 사용된다.
    
    이후에 타입스크립트가 자체적으로 생성자 체크 후 타입으로 변환을 해주기 때문에( 타입 좁히기) 클래스를 타입으로도 사용할 수 있게 된다.
    
    ```jsx
    function calculateVolume(shape: unknown) {
      if (shape instanceof Cylinder) {
        // instanceof는 런타임에 실행됨
        // 여기서 Cylinder는 "값"(생성자 함수)
        
        shape; // 타입스크립트가 여기서 shape의 타입을 Cylinder로 좁혀줌
        shape.radius; // 이제 radius 접근 가능
      }
    }
    ```
    

### 타입과 값에서 쓰임이 다른 typeof

```jsx
type T1 = typeof p; // 타입은 Person
type T2 = typeof email;
// 타입은 (p: Person, subject: string, body: string) => Response

const v1 = typeof p; // 값은 "object"
const v2 = typeof email; // 값은 "function"
```

타입의 관점에서, `typeof` 는 값을 읽어서 **타입스크립트 타입**을 반환한다. 그러나 값의 관점에서 `typeof` 는 **자바스크립트 런타임의 `typeof` 연산자**가 된다. 이는 타입스크립트 타입과는 전혀 다르다.

자바스크립트의 런타임 타입 시스템은 타입스크립트의 정적 타입 시스템보다 훨씬 간단하다.

타입스크립트의 타입은 종류가 무수히 많지만, JS는 단 6개의 런타임 타입만이 존재한다.

`Cylinder` 의 예처럼 `class` 는 값과 타입 두 가지로 모두 사용되기 때문에 `class` 에 대한 `typeof` 는 상황에 따라 다르게 동작한다.

```jsx
const v = typeof Cylinder; // 값이 "function"
type T = typeof Cylinder; // 타입이 typeof Cylinder

declare let fn: T;
const c = new fn(); // 타입이 Cylinder
// 첫번째 줄에선 함수로 적용
```

JS에서 `class` 는 실제 함수로 구현되기 때문에 첫번째 코드는 `function` 이 된다. (생성자 함수)

```jsx
type C = InstanceType<typeof Cylinder>; // 타입이 Cylinder
```

`InstanceType` 제너릭을 사용해 생성자 타입과 인스턴스 타입을 전환할 수도 있다.

### 타입의 속성을 얻을 때는 속성 접근자([ ])를 사용하자

속성 접근자인 [ ] 는 타입으로 쓰일 때도 동일하게 동작하지만, `obj['field']` 와 `obj.field` 는 값이 동일하더라도 타입은 다를 수 있다. 따라서 타입의 속성을 얻을 때는 반드시 첫번째 방법을 사용해야 한다.

```jsx
const first: Person["first"] = p["first"]; // 또는 p.first
// const ~first, p['first'] : 값
// :Person['first'], p['first'] : 타입
```

`Person['first']` 는 타입 맥락(:) 뒤에 쓰인 타입이다. 인덱스 위치에는 유니온 타입과 기본형 타입을 포함한 어떠한 타입이든 사용할 수 있다.

```jsx
type PersonEl = Person["first" | "last"]; // 타입은 string
type Tuple = [string, number, Data];
type TupleEl = Tuple[number]; // 타입은 string | number | Date
```

### 타입 공간과 값 공간의 혼동에 주의하자

```jsx
function email(options: { person: Person; subject: string; body: string }) {
  // ...
}
```

JS에서 객체 내의 각 속성을 로컬 변수로 만들어 주는 구조 분해 할당을 사용할 수 있다.

```jsx
function email({ person, subject, body }) {
  // ...
}
```

TS에서 구조 분해 할당을 사용하면, 오류가 발생한다.

```jsx
function email({
    person: Person,
        // ~ 바인딩 요소 'Person'에 암시적으로 'any' 형식이 있습니다.
    subject: string,
		    // ~ 'string' 식별자가 중복되었습니다.
		    //     바인딩 요소 'string' 에 암시적으로 'any' 형식이 있습니다.
})
```

값의 관점에서 `Person` 과 `string` 이 해석되었기 때문에 오류가 발생한다.

`Person` 이라는 변수명과 `string` 이라는 이름을 가지는 두 개의 변수를 생성하려 했기 때문이다.

문제를 해결하려면 타입과 값을 구분해야 한다.

```jsx
function email({
  person,
  subject,
  body,
}: {
  person: Person;
  subject: string;
  body: string;
});
```

코드가 장황해보이지만, 매개변수에 명명된 타입을 사용하거나 문맥에서 추론되도록 잘 동작한다.

- TS 구조 분해 할당에서 오류가 발생한 이유?
    
    ```jsx
    function email({
        person: Person,    // 오류!
        subject: string,   // 오류!
    })
    ```
    
    1. TS가 `Person` 과 `string` 을 타입이 아닌, 변수 이름으로 해석하기 때문이다.
    2. 구조 분해 할당에서 `속성 : 새이름` 의 형태는 “속성의 값을 새이름이라는 변수에 할당” 한다는 의미로 동작한다.
    
    따라서 매개변수의 구조를 정의하고(`{person, subject, body}` ), 그 후 `: { ... }`  부분에서 각 속성의 타입을 정의해야 한다.
    
    ```jsx
    // 잘못된 방법
    function greet({ name: string }) {  // 'string'을 변수 이름으로 해석함
      console.log(string);  // 'name' 속성값이 'string'이라는 변수에 들어감
    }
    
    // 올바른 방법
    function greet({ name }: { name: string }) {  // name은 변수명, string은 타입
      console.log(name);
    }
    ```
