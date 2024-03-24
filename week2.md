# useReducer
- 모던 리액트 p.69
- useState와 useReducer의 예제를 비교했을 때, 혹은 직접 사용해본다면 불필요한 과정이 많고 이렇게 짤 이유가 있는지 의문스럽다. 다만 주석을 최소화하여 코드를 작성할 경우, useReducer는 useState보다 직관적인 코드를 작성할 수 있고, useCallback을 사용하지 않아도 직관적인 action을 정의할 수 있다.

## General Usage

```tsx
// noinspection JSAnnotator

type State = {
    selected: boolean;
    editable: boolean;
};

type Action = { type: 'idle' | 'select' | 'edit'; payload?: State; };

const initialState: State = {selected: false, editable: false};

function reducer(state: State, action: Action): State {
    switch (action.type) {
        case 'idle':
            return {selected: false, editable: false};
        case 'select':
            return {selected: true, editable: false};
        case 'edit':
            return {selected: true, editable: true};
            defaut:
                throw new Error('Unexpected action type ${action.type}');
    }
    ;
}

export default function App() {
    const [state, dispatch] = useReducer(reducer, initialState);

    const handleIdle = () => {
        dispatch({type: 'idle'});
    };
    const handleSelect = () => {
        dispatch({type: 'select'});
    };
    const handleEdit = () => {
        dispatch({type: 'edit'});
    };

    return (
        <div>
            <button onClick={handleIdle} data-active={!state.selected && !state.editable}>Idle</button>
            <button onClick={handleSelect} data-active={!state.selected && state.editable}>Select</button>
            <button onClick={handleEdit} data-active={state.selected && state.editable}>Edit</button>
        </div>
    )
}
```
- 예시로서 작성한 Marker의 도구 선택 알고리즘이라 가정하자, 기존의 도구 선택은 각각의 클릭 이벤트에서 직접 state를 변경해가며 도형의 이동, 수정, 삭제, 선택 여부를 결정해야했다. 하지만 useReducer를 사용한다면 dispatch에 action type을 작성하는 것 만으로 이 문제를 해결할 수 있다.
- 다만 모든 경우에서 state보다 reducer가 우위인 것은 아니다. reducer는 주로 dispatch에서 인자로 action과 payload를 받아 객체 타입의 상태를 변경하는 데에 사용된다.

```tsx
// noinspection JSAnnotator

type State = {
	select: boolean;
	edit: boolean;
};

type Action = { type: 'select' | 'edit'; payload?: State; };

const initialState: State = { selected: false, editable: false };

function reducer(state: State, action: Action): State {
	switch(action.type) {
		case 'select': return { ...state, select: action.payload ?? !state.select };
		case 'edit': return { ...state, edit: action.payload ?? !state.edit };
		defaut:
			throw new Error('Unexpected action type ${action.type}');
	};
}

export default function App() {
	const [state, dispatch] = useReducer(reducer, initialState);

	const handleSelect = () => {
		dispatch({ type: 'select' });
	};
	const handleEdit = () => {
		dispatch({ type: 'edit' });
	};

	return (
		<div>
			<button onClick={handleSelect}>Select</button>
			<button onClick={handleEdit}>Edit</button>
		</div>
	)
}
```
- 위는 이전 예제를 단축한 것으로, action.type을 통해 어떤 속성을 변경할 지 선택하고, payload를 통해 지정된 값으로 변경하는 것이다.

## Redux
- useReducer처럼 action, payload, dispatch를 자주 사용하는 라이브러리는 상태관리 라이브러리인 Redux가 있다. 또한 Ducks Pattern이라고 전용 Redux 작성 패턴도 있다.
- Redux 역시 전역으로 선언된 Root에 dispatch를 통해 상태를 변화시킨다. 다만 데이터를 직접 삽입하는 것이 아닌 사전에 정의된 action을 통해 이를 수행한다.

# SSR
- 모던 리액트 p.252
## SSR이 나타나기까지
- Javascript는 초기엔 그저 HTML을 보조하기 위한 수단으로 등장했을 뿐이었다. 그러나 인터넷 환경이 고도화되면서 웹페이지는 다양한 기능을 수행해야했다. 그래서 등장한 것이 LAMP 스택(Linux, Apach, MySQL, PHP)이었다.
- 그리고 AngularJS를 시작으로 SPA를 기반으로 한 JAM 스택(Javascript, API, Markup)이다. 서버에서 정적으로 웹페이지를 구성한 후 API를 통해 서버와 상호작용하면서 백엔드와 프론트엔드의 확장성을 강화시키는 구조였다.
- 그러나 모바일 환경이 주류가 되고, 그에 따라 웹페이지가 감당불가능한 수준으로 많은 동작을 수행해야하면서 발생하는 문제가 있었다. SPA 특성 상 초기 로딩시간이 길어지는데, 많은 동작은 더 긴 로딩시간으로 되돌아왔다. 하나의 웹 애플리케이션에서 초기로딩에 걸리는 시간은 평균 15초로 매우 길어졌다.
- 이 문제를 해결하기 위해 나타난 것이 서버 사이드 렌더링이다.

## SSR의 장단점
### 장점
- 최초 페이지 진입이 비교적 빠르다
- SEO 최적화와 메타 데이터 제공이 쉽다
- CLS(누적레이아웃이동)이 적다
- 사용자의 디바이스 성능에 비교적 자유롭다
- 보안이 좀 더 안전하다
### 단점
- 코드 작성시 서버를 고려해야하므로 CSR보다 비교적 난이도가 높다
- 서버의 성능에 좌우되기에 적절한 서버를 구축해야한다
- 느린 작업으로 인한 서비스 지연 문제
### CSR과 SSR이 모두 필요한 이유
- SSR은 만능이 아니다.
- 현대의 SSR은 SPA와 SSR의 장점을 모두 가져오긴 했으나, 어디까지나 CSR에 비해 비교우위가 있을 뿐, 절대적 우위를 가지고 있는 것은 아니다.
- API 통신이 잦고 레이아웃이 서버에 의존적인 경향을 가진다면, 예를들어 쇼핑몰과 같은 서버측 데이터를 통해 레이아웃을 렌더링하는 경우엔 SSR이 이득일 수 있겠다. 그러나 클라이언트 측에서 대부분의 작업이 이루어지며 서버의 역할이 크게 축소된 경우 SSR의 이점을 살릴 수 없다.
- 또한 서버의 작업 크기가 큰 경우 렌더링에 영향을 주기에 이 경우엔 CSR에 비교열세다.

## SSR관련 React API
- 다만 이런 함수들을 이용해 직접 React를 SSR로 구현하는 것보단 NextJS를 사용할 것을 권장한다.
### renderToString과 renderToNodeStream
- renderToString은 JSX로 작성된 코드에서 HTML 요소만 string 형태로 뽑아낸 것을 의미한다. 이를 통해 당장 상호작용은 불가능하지만, 적어도 javascript가 로딩이 될때까지 서버측에서 렌더링된 화면을 사용자에게 제공할 수 있다.
- renderToNodeStream은 return 형태가 string이 아닌 ReadableStream이다. 이것은 NodeJS에서 사용하는 타입으로 NodeJS로 작성된 서버에서만 사용할 수 있다. string에 비해 좋은 점은 HTML의 크기가 클 경우 chunk로 쪼개어 사용자의 화면이 단순히 흰 화면으로 표시되는 시간을 비교적 줄여줄 수 있다.
### renderToStatic
- React 요소 없이 오직 HTML 요소만 return 하는 함수다. String과 NodeStream을 return할 수 있는데 역할은 같다. 다만 이걸로 return된 HTML 요소들은 hydrate를 시킬 수 없어, window 이벤트를전혀 사용할 수 없다.
### hydrate
- hydrate는 HTML 콘텐츠에 자바스크립트 핸들러나 이벤트를 붙이는 역할을 한다.

# Next.js
- 모던 리액트 p.295
- v13 내용이며, 현재 v14는 v13과는 괴리된 체계를 사용하고 있기에 어디까지나 참고용으로 읽을 것
## scss variable export
```scss
// button.module.scss
$primary: blue;
:export {  
  primary: $primary;  
}
```
```tsx
// Button.tsx
import styles from './style.module.scss';  
  
const Button = () => {  
    return (  
        <button style={{background: styles.primary}}>  
            button  
        </button>  
    )  
}  
  
export default Button;
```
- scss/sass module 관련 독특한 문법을 발견해서 정리함