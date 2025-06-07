# Item02

# Item.02 타입스크립트 설정 이해하기

요약

- 타입스크립트 컴파일러는 언어의 핵심 요소에 영향을 미치는 몇 가지 설정을 포함하고 있다.
- 타입스크립트 설정은 커맨드 라인을 이용하기보다는 `tsconfig.json` 을 사용하는 것이 좋다.
- 자바스크립트 프로젝트를 타입스크립트로 전환하는게 아니라면 `noImplicitAny` 를 설정하는 것이 좋다.
- “undefined는 객체가 아닙니다” 같은 런타임 오류를 방지하려면 `strictNullChecks` 를 설정하는 것이 좋다.
- 타입스크립트에서 엄격한 체크를 하고 싶다면 `strict` 설정을 고려하자.

---

### 가급적 TS 설정 파일을 이용하자

```jsx
function add(a,b) {
	return a+b;
}
add(10,null);
```

- 이 코드가 타입 체커를 통과하는지 확인하려면
    1. `tsc --noImplicitAny program.ts`  (`tsc --init` 으로 생성)
    2. `tsconfig.json` 설정
        
        ```json
        {
        	"comilerOptions": {
        		"noImplicitAny": true
        	}
        }
        ```
        
- 어디서 소스 파일을 찾을지, 어떤 종류의 출력을 생성할지 제어하는 내용이 대부분이나, 언어 자체의 핵심 요소를 제어하는 설정도 있다. 이는 대부분의 언어에서는 허용하지 않는 고수준 설계 설정임.
- 어떻게 설정하냐에 따라 완전히 다른 언어처럼 느껴질 수 있다.

### noImplicitAny

- `noImplicitAny` 는 변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어
    
    ```jsx
    function add(a,b) return a+b;
    
    (hover) -> function add(a:any, b:any): any
    ```
    
    - any 타입을 매개변수에 사용하면 타입 체커는 무력해진다.
    - 유용하지만 매우 주의해 사용해야 한다.
- any를 코드에 넣지 않으면 any 타입으로 간주된다. 그러나, `noImplicitAny` 가 설정되었다면 오류가 발생한다. 이는 아래와 같이 `:any` 를 선언하거나 타입을 명시해 해결한다.
    
    ```jsx
    function add(a:number, b:number) {
    	return a+b;
    }
    ```
    
- 타입스크립트는 타입 정보를 가질 때 가장 효과적이기 때문에, 되도록이면 `noImplicitAny` 를 설정해야 한다.
- 새 프로젝트를 시작한다면 처음부터 설정해 코드 가독성과 개발 생산성을 높일 수 있다.
- `noImplicitAny` 해제는 JS로 되어있는 기존 프로젝트를 타입스크롭트로 전환하는 상황에만 필요하다.

### stringNullChecks

- null과 undefined가 모든 타입에서 허용되는지 확인하는 설정이다.
    
    ```jsx
    const x: number = null;
    // 설정x -> 정상, null은 유효한 값입니다.
    // 설정o -> 'null' 형식은 'number' 형식에 할당할 수 없습니다.
    ```
    
- 만약 null을 허용하려고 한다면, 의도를 명시적으로 드러냄으로써 오류를 고칠 수 있다.
    
    ```jsx
    const x: number | null = null;
    ```
    
- null을 허용하지 않으려면, 이 값이 어디서부터 왔는지 찾아야하고, null을 체크하는 코드나 단언문(assertion)을 추가해야 한다.
    
    ```jsx
    const el = document.getElementById('status');
    el.textContent = 'Ready';
    	// ~~ 개체가 'null'인 것 같습니다.
    	
    if (el) {
    	el.textContent = 'Ready'; // 정상, null은 제외됩니다.
    }
    el!.textContent = 'Ready'; // 정상, el이 null이 아님을 단언합니다.
    ```
    
- `striectNullChecks` 는 null, undefined 관련 오류를 잡아 내는 데에 많은 도움을 주지만, 코드 작성을 어렵게 한다. 따라서, 새 프로젝트가 아니라면 굳이 설정하지 않아도 상관없다.
- `strictNullChecks` 를 설정하려면, `noImplicitAny` 를 먼저 설정해야 한다.
