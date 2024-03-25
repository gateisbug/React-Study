# Hooks

## useMemo, useCallback, memo
#### usage
```jsx
const length = useMemo(() => {
	return text.length;
}, [text])

const handler = useCallback((e) => {
	setText(e.target.handler);
}, [])

const Input = memo(() => {
	return <input />
})
```
#### Memoization
- 연산의 결과를 저장하여 변경이 없다면 연산을 진행하지 않는 것을 의미한다. useMemo는 값을, useCallback은 함수를, memo는 컴포넌트를 메모이제이션하는데 주로 사용한다. 연산은 렌더링을 지연시키는 효과가 있고, 이는 UX에 걸림돌로 작용한다. 따라서 메모이제이션은 성능 최적화에 긍정적인 영향을 주는 것으로 보인다.
- 다만 메모이제이션 역시 연산의 과정을 필요로 한다. 메모이제이션 과정과 연산 과정의 시간을 비교하여 더 효과적인 방법을 사용하는 것을 권장하지만, 명확한 지침은 따로 없기 때문에 개발자의 경험 따라 다양한 의견이 제시된다.
	- 비용이 많이 드는 계산: API 통신으로 전달된 값을 가공하는 경우, 복잡한 연산이 필요한 경우 등
	- 자식 컴포넌트로 전달하는 prop
	- hooks에서 자주 호출되는 경우나 무한루프가 발생하는 경우
#### memo
- memo는 렌더링이 자주 일어나는 컴포넌트를 메모이제이션할 때 사용한다. 렌더링은 props가 변경되거나 상태 변화가 있는 경우 트리거되는 데, 매 회 변경마다 리소스를 소모하는 데 이를 줄일 수 있는 방법이다.
	- 부모 컴포넌트는 자주 변하는데 비해, 변화가 자주 일어나지 않는 컴포넌트라면 memo로 감싸는 것도 방법이다. 예를 들어 Table과 같은 경우.
	- props가 자주 변경되는 경우엔 필요 없다.
	- 간단하고 작은 컴포넌트엔 필요 없다
	- 기본 산수나 간단한 배열 조작은 필요 없다
```jsx
const Child = memo(({ name, tell }) {
	console.log('Rendering: Child Component');

	return (
		<div style={{ border: '2px solid powderblue', padding: '10px' }}>
			<h3>Children</div>
			<p>name: {name}</p>
			<button onClick={tell}>tell</button>
		</div>
	)
});

function App() {
	const [parentAge, setParentAge] = useState(0);

	const incrementParentAge = () => {
		setParentAge(parentAge + 1);
	}

	console.log('Rendering: Parent Component');

	const tell = useCallback(() => {
		console.log('call son');
	}, []);

	return (
		<div style={{ border: '2px solid navy', padding: '10px' }}>
			<h1>Parent</h1>
			<p>age: {parentAge}</p>
			<button onClick={incrementParentAge}>time gone</button>
			<Child name='john' tell={tell} />
		</div>
	)
}
```
## useLayoutEffect
#### usage
```jsx
useLayoutEffect(() => {
	const handler = () => {};
	window.addEventListener('resize', handler);
	return () => {
		window.removeEventListener('resize', handler);
	}
}, [])
```
#### Layout Effect
- 리액트는 다음과 같은 과정으로 작동한다: 렌더링 -> useLayoutEffect -> Painting -> useEffect
- 가상DOM이 모두 구성되고 나서 호출 가능한 사이드이펙트함수다. useEffect와 달리 그리는 과정에선 호출되지 않는다.
- 다만 useLayoutEffect에서 useEffect의 의존성 상태를 변화시키는 경우 Painting 이전에 useEffect를 호출 할 수 있다. (비권장)
```jsx
function getNumbers() {
	return [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15];
}

function App() {
	const [numbers, setNumbers] = useState([]);
	const ref = useRef(null);
	
	useEffect(() => {
		const nums = getNumbers();
		setNumbers(nums);
	}, []);

	useLayoutEffect(() => {
		if(numbers.length === 0) return;
		ref.current.scrollTop = ref.current.scrollHeight;
	}, [numbers])
	
	return (
		<div
			ref={ref}
			style={{
				height: "300px",
				border: "1px solid blue",
				overflow: "scroll",
			}}
		>
			{numbers.map((number, idx) => (
				<p key={idx}>{number}</p>
			))}
		</div>
	)
}
```

## 참고자료
- [성능 하면 빠질 수 없는 메모이제이션, 네가 궁금해 (naver.com)](https://d2.naver.com/helloworld/9223303)
- [useEffect는 종종 페인트(paint) 이전에 동작합니다](https://velog.io/@lky5697/unintentional-layout-effect)
- [useEffect와 useLayoutEffect의 차이 (howdy-mj.me)](https://www.howdy-mj.me/react/useEffect-and-useLayoutEffect)
- [React Hooks에 취한다 - useLayoutEffect 누가 안알려주면 절대 모르는 훅](https://youtu.be/Svu0E-LbA4o?si=qhGUgsHPUfyRE--t)
- [React.memo로 컴포넌트 최적화하기 (ft. useMemo, useCallback)](https://youtu.be/oqUgcxwrnSY?si=K5IidB5_xmoxF00A);
