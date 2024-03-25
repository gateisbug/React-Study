# Redux Saga
`redux-saga`는 순수하지 않은 `fetching`이나 브라우저의 캐시에 접근하는 등의 `side-effect`들을, 더 쉽고 좋게 만들기 위한 라이브러리입니다. 사가는 사이드 이펙트만을 담당하는 별도의 쓰레드와 같은 것으로 보면 됩니다. 그리고 이 쓰레드를 실행하거나, 멈추거나, 취소할 수 있게 만듭니다.  
리덕스 사가는 ES6의 문법인 `function*`을 사용하며, 비동기 흐름을 마치 동기식 자바스크립트 코드처럼 보이게 만듭니다. `redux-thunk`와 달리 비동기 흐름을 더 쉽게 테스트 할 수 있고, 액션을 순수하게 유지할 수 있습니다.

## function*
```javascript
function* idMaker(){
  var index = 0;
  while(index < 3)
    yield index++;
}

var gen = idMaker();

console.log(gen.next()); // { value: 0, done: false }
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: undefined, done: false }
```
`Generator Function`은 함수 실행의 중단과 재개가 가능한 함수입니다. 함수 실행 시 `Iterator` 객체를 반환하게되고, 이 객체에서 `next()`를 호출하게 되면, `yield`문을 만날떄 까지 함수가 실행되고, `yield`의 값이 반환됩니다. `next()`를 다시 실행하게 되면 중단되었던 지점에서 재개되어 다음의 `yield`를 만날 때까지 실행됩니다. 더이상 `yield`가 없거나, `return()` 또는 `throw()` 메소드가 실행되면 종료됩니다.

## Redux Saga 기본구조
```javascript
// src/redux/store.js
import createSagaMiddleware from "redux-saga";
const sagaMiddleware = createSagaMiddleware();
const store = createStore(reducer, applyMiddleware(thunk, promise, sagaMiddleware));
```
리덕스 사가를 설정하기 위해선 `createSagaMiddleware()`를 실행해주어야 합니다. 그리고 그 결과를 `applyMiddleware()`의 인자로 추가해야 합니다. 이렇게 하면 미들웨어 설정이 끝나게 됩니다.

```javascript
// src/redux/modules/users.js
import { put, delay, call } from 'redux-saga/effects';

function* getUsersSaga(action) {
  try {
    yield put(getUsersStart()); // 액션을 dispatch
    yield delay(2000); // 2초 대기
    // 1번째 인자로 함수의 이름, 2번째 인자로 함수의 매개변수를 입력
    const res = yield call(axios.get, 'https://api.github.com/users');
    // 입력된 'https://api.github.com/users'는 필요하다면 action.payload로 전달받을 수 있습니다.
    yield put(getUsersSuccess(res.data));
  } catch (error) {
    yield put(getUsersFail(error));
  }
}
```
사이드 이펙트가 발생하는 로직을 사가함수로 만들었습니다. 여기서 특징은 리덕스 사가의 이펙트들을 `yield`로 감싸주었다는 점입니다. 이로써 `redux-thunk`와 달리 외부의 함수를 받지 않고도 비동기 로직을 처리할 수 있는 순수함수를 만들었습니다.

```javascript
// src/redux/modules/users.js
import { takeEvery } from 'redux-saga/effects';

const GET_USERS_SAGA_START = `${prefix}/GET_USERS_SAGA_START`;
export function* usersSaga() {
  yield takeEvery(GET_USERS_SAGA_START, getUsersSaga)
}
```
`GET_USERS_SAGA_START` 액션이 디스패치 되면 `getUsersSaga`사가 함수를 실행할 수 있도록, 미들웨어에 등록할 준비가 끝났습니다.

```javascript
// src/redux/modules/rootSaga.js
import { all } from 'redux-saga/effects';
import { usersSaga } from "./users";

export default function* rootSaga() {
  yield all([usersSaga()]);
}
```
위에서 만든 `usersSaga` 뿐만 아니라, 다른 사가 함수들도 `rootSaga`에 모아서 미들웨어에 포함시킬 수 있습니다.

```javascript
// src/redux/store.js
import rootSaga from "./modules/rootSaga";
sagaMiddleware.run(rootSaga);
```
`rootSaga`를 `sagaMiddleware.run(rootSaga)`로 미들웨어에 등록하면 사용할 준비를 마친 것입니다.

```javascript
// src/redux/modules/users.js
export function getUsersSagaStart() {
  return {
    type: GET_USERS_SAGA_START,
  };
}
```
사가 함수를 실행할 액션 생성함수를 만들어줍니다.

```javascript
// src/containers/Container_saga.js
export default function Container_saga() {
  const users = useSelector(state => state.users.data);
  const dispatch = useDispatch();
  const getUsers = useCallback(() => {
    dispatch(getUsersSagaStart());
  }, [dispatch]);

  return <List users={users} getUsers={getUsers} />
}
```
`redux-thunk`처럼 액션을 디스패치해주면 사가 함수가 실행됩니다. `redux-thunk`에 비해 로직은 다소 복잡할 수 있습니다. 다만, thunk와 달리 순수함수로 동작하고, 흐름을 제어하는 것이 thunk보다 쉽다는 장점이 있습니다.

## Redux Saga Effects
리덕스 사가에서 주로 사용하는 이펙트는 `delay`, `put`, `call`, `takeEvery`, `takeLatest`, `all`입니다.
```javascript
yield delay(ms);
```
`delay()`는 인자로 받은 `ms`만큼 실행을 지연시켜주는 이펙트 입니다.

```javascript
yield put(action(...args))
```
`put()`은 인자로 받은 `action(...args)`을 `dispatch`하는 이펙트 입니다.

```javascript
yield call(api.func, action.payload);
```
`call()`은 인자로 함수와 함수의 매개변수를 받아 실행시켜주는 이펙트입니다. `fetching`과 같은 비동기 로직을 실행하는데 주로 사용됩니다.

```javascript
yield takeEvery(actionType, sagaFunction);
```
`takeEvery()`는 dispatch되는 모든 actionType에 대해 sagaFunction을 실행시켜주는 이펙트입니다.

```javascript
yield takeLatest(actionType, sagaFunction);
```
`takeLatest()`의 기본적인 동작은 `takeEvery`와 동일합니다. 다만, 같은 actionType이 dispatch될 경우 기존의 작업을 취소하고 마지막으로 실행된 작업에 대해서만 sagaFunction을 실행합니다.

```javascript
yield all([...effects])
```
`all()`은 인자로 받은 제너레이터 함수의 배열을 병행적으로 동시에 실행시키고 전부 resolve 될 때까지 기다리는 이펙트입니다.

## 참고자료
- [redux-saga Official](https://mskims.github.io/redux-saga-in-korean/)
- [redux-saga 가 해결하는 문제](https://min9nim.vercel.app/2020-04-23-redux-saga/)
- [Redux Thunk vs Redux Saga](https://dev-recruiting.ringleplus.com/4850db36-7f98-4b27-8112-e152a1a2ab5b)
- [Redux-Thunk vs Redux-Saga를 비교해 봅시다!](https://velog.io/@dongwon2/Redux-Thunk-vs-Redux-Saga를-비교해-봅시다-)
- [redux-saga의 주요함수](https://sustainable-dev.tistory.com/94)
