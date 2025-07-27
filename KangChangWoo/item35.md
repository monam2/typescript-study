# Item 35

## item.35 데이터가 아닌, API와 명세를 보고 타입 만들기

*요약*

- 코드의 구석 구석까지 타입 안정성을 얻기 위해 API 또는 데이터  형식에 대한 타입 생성을 고려해야 한다.
- 데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있기 때문에 데이터보다는 명세로부터 코드를 생성하는 것이 좋다.

---

이 장의 다른 아이템들에서는 타입을 잘 설계하면 어떠한 이점이 있는지, 반대로 설계가 잘못되면 무엇이 잘못될 수 있는지에 대해서 다루었다. 잘 설계된 타입은 타입스크립트 사용을 즐겁게 해 주는 반면, 잘못 설계된 타입은 비극을 불러온다. 이러한 양면성 때문에 타입 설계를 잘 해야 한다는 압박감이 느껴질 수 있다. 이런 상황에서 타입을 직접 작성하지 않고 자동으로 생성할 수 있다면 매우 유용할 것이다.

파일 형식, API, 명세(specification) 등 우리가 다루는 타입 중 최소한 몇 개는 프로젝트 외부에서 비롯된 것이다. 이러한 경우는 타입을 직접 작성하지 않고 자동으로 생성할 수 있다. 여기서 핵심은, 예시 데이터가 아니라 명세를 참고해 타입을 생성한다는 것이다.

명세를 참고해 타입을 생성하면 타입스크립트는 사용자가 실수를 줄일 수 있게 도와준다. 반면에 예시 데이터를 참고해 타입을 생성하면 눈앞에 있는 데이터들로만 고려하게 되므로 예기치 않은 곳에서 오류가 발생할 수 있다.

Item31에서 사용한 `Feature` 의 경계 상자를 계산하는 `calculateBoundingBox` 함수를 구현해보자.

```jsx
function calculateBoundingBox(f: Feature): Boundingox | null {
	let box: BoundingBox | null = null;
	
	const helper = (coords: any[]) => {
		// ...
	};
	
	const {geometry} = f;
	if (geometry) {
		helper(geometry.coordinates);
	}
	
	return box;
}
```

`Feature` 타입은 명시적으로 정의된 적이 없다. `focusOnFeature` 함수 예제를 사용해 작성해 볼 수 있다. 그러나 공식 `GeoJSON` 명세를 사용하는 것이 더 낫다. 다행히도 DefinitelyTyped에는 이미 타입스크립트 타입 선언이 존재한다. 따라서 다음과 같이 익숙한 방법을 이용해 추가할 수 있다.

```jsx
$ npm install --save-dev @types/geojson
+ @types/geojson@7946.0.7
```

`GeoJSON` 선언을 넣는 순간, 타입스크립트는 오류를 발생시킨다.

```jsx
import {Feature} from 'geojson';

function calculateBoundingBox(f: Feature): BoundingBox | null {
	let box: BoundingBox | null = null;
	
	const helper = (coords: any[]) => {
		// ...
	};
	
	const {geometry} = f;
	if (geometry) {
		helper(geometry.coordinates);
									// ~~~~~~~~~~ 
									// 'Geometry' 형식에 'coordinates' 속성이 없습니다.
									// 'GeometryCollection' 형식에
									// 'coordinates' 속성이 없습니다.
	}
	
	return box;
}
```

`geometry` 에 `coordinates` 속성이 있다고 가정한 게 문제이다. 이러한 관계는 점, 선, 다각형을 포함한 많은 도형에서는 맞는 개념이다. 그러나 `GeoJSON` 은 다양한(heterogeneous) 도형의 모음인 `GeometryCollection` 일 수도 있다. 다른 도형 타입들과 다르게 `GeometryCollection` 에는 `coordinates` 속성이 없다.

`geometry` 가 `GeometryCollection` 타입인 `Feature` 를 사용해서 `calculatedBoundingBox` 를 호출하면 `undefined` 의 `0` 속성을 읽을 수 없다는 오류를 발생시킨다.

이 오류를 고치는 한 가지 방법은 다음 코드처럼 `GeometryCollection` 을 명시적으로 차단하는 것이다.

```jsx
const {geometry} = f;
if (geometry) {
	if (geometry.type === 'GeometryCollection') {
		throw new Error('GeometryCollections are not supported.');
	}
	helper(geometry.coordinates); // 정상
}
```

타입스크립트는 타입을 체크하는 방법으로 도형의 타입을 정제할 수 있으므로 정제된 타이벵 한해서 `geometry.coordinates` 의 참조를 허용하게 된다. 차단된 `GeometryCollection` 타입의 경우, 사용자에게 명확한 오류 메시지를 제공한다.

그러나 `GeometryCollection` 타입을 차단하기보다는 모든 타입을 지원하는 것이 더 좋은 방법이기 때문에 조건을 분기해서 헬퍼 함수를 호출하면 모든 타입을 지원할 수 있다.

```jsx
const geometryHelper = (g: Geometry) => {
	if (geometry.type === 'GeometryCollection') {
		geometry.geometries.forEach(geometryHelper);
	} else {
		helper(geometry.coordinates); // 정상
	}
}

const {geometry} = f;
if (geometry) {
	geometryHelper(geometry);
}
```

그동안 GeoJSON을 사용해온 경험을 바탕으로 GeoJSON의 타입 선언을 직접 작성했을 수도 있다. 아마도 직접 작성한 타입 선언에는 `GeometryCollection` 같은 예외 상황이 포함되지 않았을 테고 완벽할 수도 없다. 반면, 명세를 기반으로 타입을 작성한다면 현재까지 경험한 데이터뿐만 아니라 사용 가능한 모든 값에 대해서 작동한다는 확신을 가질 수 있다.

API 호출에도 비슷한 고려 사항들이 적용된다. API의 명세로부터 타입을 생성할 수 있다면 그렇게 하는 것이 좋다. 특히 `GraphQL` 처럼 자체적으로 타입이 정으된 API에서 잘 동작한다.

`GraphQL` API는 타입스크립트와 비슷한 타입 시스템을 사용하여, 가능한 모든 쿼리와 인터페이스를 명세하는 스키마로 이뤄진다. 우리는 이러한 인터페이스를 사용해서 특정 필드를 요청하는 쿼리를 작성한다. 예를 들어, GitHub GraphQL API를 사용해서 저장소에 대한 정보를 얻는 코드는 다음처럼 작성할 수 있다.

```jsx
query {
	repository(owner: "Microsoft", name: "TypeScript") {
		createAt
		description
	}
}
```

결과는 다음과 같다.

```jsx
{
	"data": {
		"repository": {
			"creatAt": "2014-06-17T15:28:39Z",
			"description":
				"TypeScript is a superset of JavaScript that compiles to JavaScript."
		}
	}
}
```

GraphQL의 장점은 특정 쿼리에 대한 타입스크립트 타입을 생성할 수 있다는 것이다. GeoJSON 예제와 마찬가지로 GraphQL을 사용한 방법도 타입에 `null` 이 가능한지 여부를 정확하게 모델링할 수 있다.

다음 예제는 GitHub 저장소에서 오픈 소스 라이선스를 조회하는 쿼리이다.

```jsx
query getLicense($owner: String!, $name: String!) {
	repository(owner: $owner, name: $name) {
		description
		licenseInfo {
			spdxId
			name
		}
	}
}
```

`$owner` 와 `$name` 은 타입이 정의된 GraphQL 변수이다. 타입 문법이 타입스크립트와 매우 비슷하다. `string` 은 GraphQL의 타입이다. 타입스크립트에서는 `string` 이 된다.

그리고 타입스크립트에서 `string` 타입은 `null` 이 불가능하지만 GraphQL의 `String` 타입에서는 `null` 이 가능하다. 타입 뒤의 `!` 는 `null` 이 아님을 명시한다.

GraphQL 쿼리를 타입스크립트 타입으로 변환해 주느 많은 도구가 존재한다. 그중 하나는 Apollo이다. 다음은 Apollo를 어떻게 사용하는지 보여준다.

```jsx
$apollo client:codegen \
	--endpoint https://api.github.com/graphql \
	--includes license.graphql \
	--target typescript
Loading Apollo Project
Generating query files with 'typescript' target - wrote 2 files
```

쿼리에서 타입을 생성하려면 GraphQL 스키마가 필요하다. Apollo는 `api.github.com/graphql` 로부터 스키마를 얻는다. 실행의 결과는 다음과 같다.

```jsx
export interface getLicense_repository_licenseInfo {
	__typename: "License";
	/** Short identifier specified by <https://spdx.org/lecenses> */
	spdxId: string | null;
	/** The license full name specified by <https://spdx.org/licenses> */
	name: string;
}

export interface getLicense_repository {
	__typename: "Repository";
	/** The description of the repository. */
	description: string | null;
	/** The license associated with the repository */
	licenseInfo: getLicense_repository_licenseInfo | null;
}

export interface getLicense {
	/** Lookup a given repository by the owner and repository name. */
	repository: getLicense_repository | null;
}

export interface getLicenseVariables {
	owner: string;
	name: string;
}
```

주목할 만한 점은 다음과 같다.

- 쿼리 매개변수(`getLicenseVariables` )와 응답(`getLicense` ) 모두 인터페이스가 생성되었다.
- `null` 가능 여부는 스키마로부터 응답 인터페이스로 변환되었다. `repository` , `dscription` , `licenseInfo` , `spdxId` 속성은 `null` 이 가능한 반면, `name` 과 쿼리에 사용된 변수들은 그렇지 않다.
- 편집기에서 확인할 수 있도록 주석은 JSDoc으로 변환되었다. 이 주석들은 GraphQL 스키마로부터 생성되었다.

자동으로 생성된 타입 정보는 API를 정확히 사용할 수 있도록 도와준다. 쿼리가 바뀐다면 타입도 자동으로 바뀌며 스키마가 바뀐다면 타입도 자동으로 바뀐다. 타입은 단 하나의 원천 정보인 `GraphQL` 스키마로부터 생성되기 때문에 타입과 실제 값이 항상 일치한다.

만약 명세 정보나 공식 스키마가 없다면 데이터로부터 타입을 생성해야 한다. 이를 위해 quicktype같은 도구를 사용할 수 있다. 그러나 생성된 타입이 실제 데이터와 일치하지 않을 수 있다는 점을 주의해야 한다. 예외적인 경우가 존재할 수 있다.

우리는 이미 자동 타입 생성의 이점을 누리고 있다. 브라우저 DOM API에 대한 타입 선언은 공식 인터페이스로부터 생성되었다. 이를 통해 복잡한 시스템을 정확히 모델링하고 타입스크립트가 오류나 코드상의 의도치 않은 실수를 잡을 수 있게 한다.
