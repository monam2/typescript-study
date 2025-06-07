# Item05

# Item.05 any 타입 지양하기

*요약*

- any 타입을 사용하면 타입 체커와 타입스크립트 언어 서비스를 무력화시킨다.
- any 타입은 진짜 문제점을 감추며, 개발 경험을 나쁘게 하고, 타입 시스템의 신뢰도를 떨어뜨린다.

---

### any에는 타입 안정성이 없다.

```jsx
let age: number;
age = '12';
// ~~~ '"12"' 형식은 'number' 형식에 할당할 수 없습니다.
age = '12' as any; // OK

age += 1; // 런타임에 정상, age는 "121"
```

- as any를 사용하면 string 타입을 할당할 수 있게 된다. 타입 체커는 선언에 따라 number 타입으로 판단하므로 타입에 혼돈이 발생한다.

### any는 함수 시그니처를 무시한다.

- 함수 시그니쳐는 호출-반환의 관계에서 함수가 갖는 약속이다. any를 사용하면 이런 약속을 어기게 된다.

### any 타입에는 언어 서비스가 적용되지 않는다.

- 타입스크립트는 어떤 심벌에 타입이 있을 경우 자동완성 기능과 적절한 도움말을 제공한다.
- 그러나 any 타입인 심벌을 사용하면 타입스크립트의 도움을 받을 수 없다.
- 타입스크립트의 모토는 **‘확장 가능한 자바스크립트’** 이다. 확장의 중요한 부분은 바로 타입스크립트 경험의 핵심 요소인 언어 서비스이다. 이는 곧 개발자의 생산성 향상으로 이어진다.

ㄹ

### any 타입은 코드 리팩터링 시 버그를 감춘다.

```jsx
interface ComponentProps {
	onSelectItem: (item: any) => void;
}

function renderSelector(props: ComponentProps) { /* ... */}

let selectedId: number = 0;

function handleSelectItem(item: any) {
	selectedId = item.id;
}

renderSelector({onSelectItem: handleSelectItem});

// ComponentProps의 시그니처를 변경
interface ComponentProps {
	onSelectItem: (id: number) => void;
}
```

- `handleSelectItem` 은 any 매개변수를 받는다. 따라서 id를 전달받아도 문제가 없다고 나온다.
- id를 전달받으면, 타입 체커를 통과함에도 런타임에는 오류가 발생한다. any가 아닌, 구체적인 타입을 사용했다면 오류를 발견했을 것이다.

### any는 타입 설계를 감춘다.

- any는 객체를 정의할 때 상태 객체의 설계를 감춘다.
- 깔끔하고 명료한 코드 작성을 위해 제대로 된 타입 설계는 필수인데, any 타입은 타입 설계를 불분명하게 한다.

### any는 타입 시스템의 신뢰도를 떨어뜨린다.

- 타입 체커는 실수를 잡아주고 코드의 신뢰성을 보장한다. 그러나 런타임에 타입 오류를 발견하게 된다면 타입 체커를 신뢰할 수 없다.
