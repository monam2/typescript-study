# Item04

# Item.04 구조적 타이핑에 익숙해지기

*요약*

- 자바스크립트가 덕 타이핑 기반이고 타입스크립트가 이를 모델링하기 위해 구조적 타이핑을 사용한다.
- 어떤 인터페이스에 할당 가능한 값이라면 타입 선언에 명시적으로 나열된 속성들을 가지고 있다. 타입은 ‘봉인’ 되어 있지 않다.
- 클래스 역시 구조적 타이핑 규칙을 따른다는 것을 명심해야 한다. 클래스의 인스턴스가 예상과 다를 수 있다.
- 구조적 타이핑을 사용하면 유닛 테스팅을 손쉽게 할 수 있다.

---

### 자바스크립트는 본질적으로 덕 타이핑(duck typing) 기반이다.

- 타입스크립트는 매개변수 값이 요구사항을 만족한다면 타입이 무엇인지 신경쓰지 않는 동작을 그대로 모델링한다.
- 즉, 함수의 매개변수 값이 모두 주어진다면, 그 값이 어떻게 만들어졌는지 신경쓰지 않고 사용한다.
- 그러나 타입 체크의 타입 이해도가 사람과 달라 예상치 못한 결과를 유발하기도 한다.
    
    ```jsx
    interface Vector2D {
    	x: number;
    	y: number;
    }
    function calculateLength(v: Vector2D) {
    	return Math.sqrt(v.x * v.x + v.y + v.y);
    }
    
    interface NamedVector {
    	name: string;
    	x: number;
    	y: number;
    }
    
    const v: NamedVector = { x:3, y: 4, name: 'Zee' };
    calculateLength(v); // 정상, 결과는 5
    ```
    
- `NamedVector` 는 number타입의 x와 y 속성이 있기 때문에 `calculateLength` 함수로 호출 가능하다.
- 위 코드에서 `Vector2D` 와 `NamedVector` 의 관계는 전혀 선언되지 않았고, 별도로 구현할 필요도 없다.
- `NameVector` 의 구조가 `Vector2D` 와 호환되기 때문에 `calculatedLength`  호출이 가능하다. 이를 구조적 타이핑(duck typing)이라 부른다.

### 구조적 타이핑이 문제를 유발하는 경우

```jsx
interface Vector3D {
	x: number;
	y: numberr;
	z: number;
}

function normalize(v: Vector3D) {
	const length = calculatedLength(v);
	return {
		x: v.x / length,
		y: v.y / length,
		z: v.z / length,
	};
}

// 그러나 이 함수는 1보다 조금 더 긴(1.41) 길이를 가진 결과를 출력한다.
> normalize({x: 3, y: 4, z: 5})
{x:0.6, y: 0.8, z: 1}
```

- `calculateLength` 는 `Vector2D`를 기반으로 연산하지만, `normalize`가 3D벡터로 연산되었다. z정규화가 무시된 것. 그런데, 타입체커가 이를 잡아내지 못했다.
- v는 Vector3D지만, 구조적 타이핑 관점에서 x와 y가 있어서 타입체커가 이를 문제로 인식하지 않는다.
