# Item 15

## Item.15 동적 데이터에 인덱스 시그니처 사용하기

*요약*

- 런타임 때까지 객체의 속성을 알 수 없을 경우에만(예를 들어 CSV 파일에서 로드하는 경우) 인덱스 시그니처를 사용하는 것이 좋다.
- 안전한 접근을 위해 인덱스 시그니처의 값 타입에 `undefined` 를 추가하는 것을 고려해야 한다.
- 가능하다면 인터페이스, `Record` , 매핑된 타입 같은 인덱스 시그니처보다 정확한 타입을 사용하는 것이 좋다.

---

자바스크립트의 장점 중 하나는 **객체를 생성하는 문법이 간단하다**는 것이다.

```jsx
const rocket = {
	name: 'Falcon 9',
	variant: 'Block 5',
	thrust: '7,607kN',
};
```

자바스크립트 객체는 문자열 키를 타입의 값에 관계없이 매핑한다. 타입스크립트에서는 타입에 ‘인덱스 시그니처’를 명시하여 유연하게 매핑을 표현할 수 있다.

```jsx
type Rocket = {[property: string]: string};
const rocket: Rocket = {
	name: 'Falcon 9',
	variant: 'Block 5',
	thrust: '7,607kN',
}; // 정상
```

`[property: string]: string` 이 인덱스 시그니처이며, 다음 세 가지 의미를 담고 있다.

- **키의 이름 : 키의 위치만 표시하는 용도. 타입 체커에서는 사용하지 않기 때문에 무시할 수 있는 참고 정보로 생각한다.**
- **키의 타입 : `string` 이나 `number` 또는 `symbol` 의 조합이어야 하지만, 보통은 `string` 을 사용한다.**
- **값의 타입 : 어떤 것이든 될 수 있다.**

이렇게 타입 체크가 수행되면 네 가지 단점이 드러난다.

- 잘못된 키를 포함해 모든 키를 허용한다. `name` 대신 `Name` 으로 작성해도 유요한 `Rocket` 타입이 된다.
- 특정 키가 필요하지 않다. `{}` 도 유효한 `Rocket` 타입이다.
- 키마다 다른 타입을 가질 수 없다. 예를 들어, `thrust` 는 `string` 이 아니라 `number` 여야 할 수도 있다.
- 타입스크립트 언어 서비스는 다음과 같은 경우에 도움이 되지 못한다. `name:` 을 입력할 때, 키는 무엇이든 가능하기 때문에 자동 완성 기능이 동작하지 않는다.

이를 결론내리면, 인덱스 시그니처는 부정확하므로 더 나은 방법을 찾아야 한다. 예를 들어 다음 경우에는 `Rocket` 이 인터페이스여야 한다.

```jsx
interface Rocket {
	name: string,
	variant: string,
	thrust_kN: number,
}

const falconHeavy: Rocket = {
	name: 'Falcon Heavy',
	variant: 'v1',
	thrust_kN: '15_200',
}
```

`thrust_kN` 은 `number`  타입이며, 타입스크립트는 모든 필수 필드가 존재하는지 확인한다. 이제 타입스크립트에서 제공하는 언어 서비스를 모두 사용할 수 있게 되었다. 자동완성, 정의로 이동, 이름 바꾸기 등이 모두 동작한다.

인덱스 시그니처는 동적 데이터를 표현할 때 사용한다. 예를 들어 CSV 파일처럼 헤더 행(row)에 열(column) 이름이 있고, 데이터 행을 열 이름과 값으로 매핑하는 객체로 나타내고 싶은 경우이다.

```jsx
function parseCSV(input: string): {[columnName: string]: string}[] {
	const lines = input.split('\n');
	const [header, ...rows] = lines;
	const headerColumns = header.split(',');
	
	return rows.map(rowStr => {
		const row: {[columnName: string]: string} = {};
		rowStr.split(',').forEach((cell, i) => {
			row[headerColumns[i]] = cell;
		});
		return row;
	});
}
```

일반적인 상황에서 열 이름이 무엇인지 미리 알 방법은 없다. 이럴 때는인덱스 시그니처를 사용한다. 반면에 열 이름을 알고 있는 특정한 상황에 `parseCSV` 가 사용된다면, 미리 선언해 둔 타입으로 단언문을 사용한다.

```jsx
interface ProductRow {
	productId: string;
	name: string;
	price: string;
}

declare let csvData: string;
const products = parseCSV(csvData) as unknown as ProductRow[];
```

물론 선언해 둔 열들이 런타임에 실제로 일치한다는 보장은 없다. 이 부분이 걱정된다면 값 타입에 `undefined` 를 추가할 수 있다.

```jsx
function safcParseCSV(
	input: string
): {[columnName: string]: string | undefined}[] {
	return parseCSV(input);
}
```

이제 모든 열의 `undefined` 여부를 체크해야 한다.

```jsx
const rows = parseCSV(csvData);
const prices: {[product: string]: number} = {};

for (const row of rows) {
	prices[row.productId] = Number(row.price);
}

const safeRows = safeParseCSV(csvData);
for (const row of safeRows) {
	prices[row.productId] = Number(row.price);
	// ~~ 'undefined' 형식을 인덱스 형식으로 사용할 수 없습니다.
}
```

물론 체크를 추가해야 하기에 작업이 조금 번거로울 수 있다. `undefined` 를 타입에 추가할지는 상황에 맞게 판단해야 한다.

연관 배열(associative array)의 경우, 객체에 인덱스 시그니처를 사용하는 대신 `Map` 타입을 사용하는 것을 고려할 수 있다. 이는 프로토타입 체인과 관련된 유명한 문제를 우회한다.

어떤 타입에 가능한 필드가 제한되어 있는 경우라면 인덱스 시그니처로 모델링하지 말아야 한다. 예를 들어 데이터에 A, B, C, D 같은 키가 있지만, 얼마나 많이 있는지 모른다면 선택적 필드 또는 유니온 타입으로 모델링하면 된다.

```jsx
interface Row1 { [column: string]: number } // 너무 광범위
interface Row2 {
	a: number;
	b?: number;
	c?: number;
	d?: number;
} // 최선

type Row3 =
	| { a: number; }
	| { a: number; b: number; }
	| { a: number; b: number; c: number;}
	| { a: number; b: number; c: number; d: number}
// 가장 정확하지만 사용하기 번거로움
```

마지막 형태가 가장 정확하지만, 사용하기에는 조금 번거롭다.

`string` 타입이 너무 광범위해서 인덱스 시그니처를 사용하는 데 문제가 있다면, 두 가지 다른 대안을 생각해 볼 수 있다.

첫 번째, `Rocord` 를 사용하는 방법이다. `Record` 는 키 타입에 유연성을 제공하는 제너릭 타입이다. 특히, `string` 의 부분 집합을 사용할 수 있다.

```jsx
type Vec3D = Record<'x' | 'y' | 'z', number>;
//Type Vec3D = {
//	x: number;
//	y: number;
//	z: number;
//}
```

두 번째, 매핑된 타입을 사용하는 방법이다. 매핑된 타입은 키마다 별도의 타입을 사용하게 해 준다.

```jsx
type Vec3D = {[k in 'x' | 'y' | 'z']: number};
//Type Vec3D = {
//	x: number;
//	y: number;
//	z: number;
//}

type ABC = {[k in 'a' | 'b' | 'c']: k extends 'b' ? string : number};
//Type ABC = {
//	a: number;
//	b: string;
//	c: number;
//}
```
