# Item06

## Item.06 편집기를 사용하여 타입 시스템 탐색하기

요약

- 편집기에서 타입스크립트의 언어 서버시를 적극 활용해야 한다.
- 편집기를 사용하면 어떻게 타입 시스템이 동작하는지, 타입스크립트가 어떻게 타입을 추론하는지 개념을 잡을 수 있다.
- 타입스크립트가 동작을 어떻게 모델링하는지 알기 위해 타입 선언 파일을 찾아보는 방법을 터득해야 한다.
    - “동작을 모델링한다”
        - 실제 코드의 작동 방식(동작)을 타입 시스템 안에 표현해둔 것
        - **실제 JS 코드가 어떻게 실행될 지를 TypeScript가 타입 정보를 통해 미리 설명하고 예측할 수 있도록 구조화했다.**

---

### 타입스크립트 사용 시 컴파일러(tsc)와 TS 서버(tsserver)를 이용할 수 있다.

- 타입스크립트를 사용하는 주 용도는 컴파일러를 쓰기 위함이지만, 타입스크립트 서버 역시 언어 서비스를 제공하기에 이를 사용하면 좋다. 이는 자동완성, 명세 검사, 검색, 리팩터링 등을 포함한다.
- 편집기는 TypeScript가 언제 타입 추론을 수행할 수 있는지에 대한 개념을 잡게 해준다.

- TS 서버란?
    
    TypeScript의 언어 서비스(Language Service)를 제공하는 서버.
    
    단독으로도 실행할 수 있지만, 대부분 IDE가 내부적으로 실행시켜 사용된다.
    
    즉, IDE(편집기)는 tsserver의 언어 서비스 기능(자동완성, 타입오류 체크 등)을 사용하는 클라이언트라고 볼 수 있다.
    
    ```jsx
    1. VSCode가 .ts 파일을 열면
    2. 내부적으로 tsserver를 실행하거나 연결한다.
    3. 사용자가 코드를 입력할 때마다 tsserver에 현재 코드 상태를 전송한다.
    4. tsserver는 타입 검사, 자동 완성 제안(suggestion) 등을 반환한다. (언어 서비스)
    5. 반환 받은 정보를 기반으로 VSCode가 사용자에게 메시지를 표시한다. (힌트, 에러, 자동완성 등)
    ```
    
- TS서버를 수동으로 실행하기
    
    TypeScript를 설치하면 `tsserver` 라는 명령어도 함께 설치된다. 이를 수동으로 직접 실행할 수 있다.
    
    이때 `tsserver` 는 표준 입력(stdin)/출력(stdout) 또는 소켓을 통해 다른 프로그램과 JSON 형태로 통신한다. 즉, `tsserver` 는 명령을 JSON으로 받고, 응답도 JSON으로 돌려주는 형식인 것.
    
    LSP(Laungauge Server Protocol)과 유사하지만, `tsserver` 는 이를 따르지 않고 JSON-RPC 스타일의 자체 프로토콜을 사용한다.
    
    `npx tsserver` 를 실행하면 아무런 메시지 없이 대기 상태에 들어간다. 이 상태에서 표준 입력으로 JSON 요청을 보내면 응답을 받을 수 있다.
    
    ```jsx
    {
      "seq": 0,
      "type": "request",
      "command": "compilerOptionsForInferredProjects",
      "arguments": {
        "options": {
          "module": "commonjs",
          "target": "es6"
        }
      }
    }
    ```
    
    위의 메시지는 `tsserver` 에 버전 정보를 요청하는 JSON 메시지이다. 이걸 `stdin` 으로 보내면 `tsserver` 는 JSON 으로 응답을 준다.
    
    ### 왜 수동으로 실행하는가?
    
    보통은 에디터가 처리하지만, 아래의 경우 수동 실행이 필요하다.
    
    - 에디터, IDE를 자체 개발 하는 경우
    - 특정 자동화 도구나 분석 도구가 타입스크립트 정보를 얻고자 할 때
    - 언어 기능을 커스터마이징하는 경우
    - LSP 서버로 말아서 tsserver를 LSP 처럼 쓰고 싶은 경우
    
    > ESLint 도 내부적으로 `tsserver` 에서 타입 정보를 얻어 분석에 활용한다.
    > 
    
    참고자료
    
    - https://github.com/microsoft/TypeScript/wiki/Standalone-Server-%28tsserver%29
    - https://code.visualstudio.com/docs/languages/typescript

### 타입스크립트의 타입 추론 살펴보기

```jsx
function logMessage(message: string | null) {
	if (message) {
		message // -> (parameter) message: string
	}	
}
```

조건문 외부에서 message의 타입은 `string | null` 이지만 내부에서는 `string` 이다.

```jsx
const foo = {
	x: [1,2,3], // (property) x: number[]
	bar: {
		name: 'Fred'
	}
}
```

현재 x의 타입은 `number[]` 인데, 튜플 타입을 갖게하기 위해선(`[number, number, number]` ) 타입 구문을 명시해야 한다.

연산자 체인(chain) 중간의 추론된 제네릭 타입을 알고 싶다면, 메서드를 살펴보면 된다.

→ 각 메서드가 어떤 타입의 값을 반환하고 있는지 궁금하다면, 해당 메서드에 마우스를 올려서 확인할 수 있다.

```jsx
restOfPath('users/123/profile') 
path.split('/').slice(1).join('/')
// (method) Array<string>.slice(start?: number, end?: number): string[]
```

`.slice()` 를 호출하는 대상(`path.split(’/’)`)이 `Array<string>` 이라는 것을 TypeScript가 추론했다는 뜻이다.

즉, `path.split('/')` 의 결과가 `string[]` 이라는 것을 타입 추론을 통해 자동으로 이해한 것.

`someData.getItems().filter(...).map(...).reduce(...)` 와 같이 체인이 길어지면 중간 결과가 어떤 타입인지 모르고 헷갈리게 된다. 그럴때 `filter, map` 같은 중간 메서드 위에 마우스를 올려 추론 타입을 확인할 수 있다. → 디버깅에서 중요하게 사용

### 편집기의 타입 오류 살펴보기

```jsx
function getElement(elOrId: string|HTMLElement | null): HTMLElement {
	if (typeof elOrId === 'object') {
		return elorId;
		// ~ 'HTMLElement | null'형식은 'HTMLELement' 형식에 할당할 수 없습니다.
	} else if (elOrId === null) {
		return document.body;
	} else {
		const el = document-getElementById(elOrId);
		return el;
		// ~ 'HTMLElement | null'형식은 'HTMLELement' 형식에 할당할 수 없습니다.
	}
}
```

첫 번째 if 분기의 의도는 `HTMLElement` 라는 객체를 골라내고자 한 것이다. 그러나, 자바스크립트에서 `typeof null` 은 `object` 이므로, `elOrId` 분기문 내에서 여전히 `null` 로 존재한다.

두번째 오류는 `document.getElementByID` 가 `null` 을 반환할 가능성이 있어서 발생했고, 첫 번째 오류와 동일하게 Null 체크를 추가하고 예외를 던져야 한다.

### 언어 서비스는 라이브러리와 라이브러리의 타입 선언을 탐색할 때 도움이 된다.

```jsx
const response = fetch('http://example.com');
```

`fetch` 의 정의로 이동하면 TS에 포함되어 있는 DOM 타입 선언인 `lib.dom.d.ts` 로 이동한다.

```jsx
declare function fetch(
	input : RequestInfo, init?: RequestInit
): Promise<Response>;
```

`fetch` 가 `Promise` 를 반환하고 두 개의 매개변수 (`input, init` )을 받는다.

```jsx
type RequestInfo = Request | string;
// Request

declare var Request: {
	prototype: Request;
	new(input: RequestInfo, init?: RequestInit): Request;
}
```

`Request` 의 타입과 값은 분리되어 모델링되어 있다. `Request Init` 을 클릭하면 `Request` 를 생성할 때 사용할 수 있는 모든 옵션이 나타난다.

```jsx
interface RequestInit {
	body?: BodyInit | null;
	cache?: RequestCache;
	credntials?: RequestCredentials;
	headers?: HeadersInit;
	// ...
}
```

타입 선언은 타입스크립트가 무엇을 하는지, 어떻게 라이브러리가 모델링되었는지, 어떻게 오류를 찾아낼지 살펴볼 수 있는 훌륭한 수단이다.
