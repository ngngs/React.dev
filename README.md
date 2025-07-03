# React 17, Redux 4, Axios 기초 가이드

---

## 1단계: JSX와 컴포넌트 구조

### JSX란?
- JavaScript 안에서 HTML처럼 작성하는 문법
- React 요소를 생성하는 문법적 설탕

```jsx
const element = <h1>Hello, world!</h1>;
```

```js
// 실제로는 다음과 같이 해석됨
React.createElement('h1', null, 'Hello, world!");
```

### 컴포넌트란?
- UI를 나누는 독립적인 작은 단위(HTML + 로직을 포함)
- 함수형 컴포넌트 예시

```jsx
function Hello(props) {
  return <h1>Hello, {props.name}!</h1>;
}
// props는 부모에 접근할 때 씀
function App() {
  return (
    <div>
      <Hello name="Alex" />
      <Hello name="Moon" />
    </div>
  );
}
```

### JSX 문법 규칙
| 항목                | 설명                          | 예시                                                             |
| ----------------- | --------------------------- | -------------------------------------------------------------- |
| 하나의 부모 태그         | JSX는 반드시 하나의 루트 엘리먼트로 감싸야 함 | ✅ `<div><h1>Title</h1></div>`<br>❌ `<h1>Title</h1><p>Text</p>` |
| JavaScript 표현식 사용 | `{}` 안에 변수나 연산식 사용 가능       | `<p>{name}</p>`                                                |
| 속성 이름은 camelCase  | HTML 속성은 자바스크립트 스타일         | `class` → `className`, `for` → `htmlFor`                       |


---

## 2단계: useState, useEffect

### useState
- 상태값을 만들고 변경하는 Hook

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
    </>
  );
}
```

### useEffect
- 컴포넌트 렌더링 이후 실행할 작업(부수 효과) 처리

```jsx
import React, { useState, useEffect } from 'react';

function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => setSeconds(prev => prev + 1), 1000);
    return () => clearInterval(interval);
  }, []);

  return <p>경과 시간: {seconds}초</p>;
}
```

---

## 3단계: Props와 컴포넌트 분리

### Props란?
- 부모가 자식에게 전달하는 읽기 전용 데이터

```jsx
function Greeting(props) {
  return <h1>안녕하세요, {props.name}님!</h1>;
}

function App() {
  return <Greeting name="Alex" />;
}
```

### 컴포넌트 분리 예

```jsx
function UserCard(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>나이: {props.age}세</p>
    </div>
  );
}

function App() {
  return (
    <>
      <UserCard name="Alex" age={29} />
      <UserCard name="Jamie" age={24} />
    </>
  );
}
```

---

## 4단계: Redux와 React 연결 (`react-redux`)

### Redux 기본

- Store: 상태 저장소
- Action: 상태 변경 신호
- Reducer: 상태 변경 함수

### React에서 Redux 사용하기

```jsx
import { useSelector, useDispatch } from 'react-redux';
import { INCREMENT } from './counterSlice';

function App() {
  const count = useSelector(state => state.count);
  const dispatch = useDispatch();

  return (
    <>
      <h1>카운터: {count}</h1>
      <button onClick={() => dispatch({ type: INCREMENT })}>+1</button>
    </>
  );
}
```

### 스토어 생성 예

```js
import { createStore } from 'redux';

const initialState = { count: 0 };

function counterReducer(state = initialState, action) {
  switch(action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    default:
      return state;
  }
}

const store = createStore(counterReducer);
export default store;
```

---

## 추가 설명: `dispatch(actions.getSome({say:1}))`

- `actions.getSome`은 액션 생성 함수로, 액션 객체를 만듭니다.
- `dispatch`는 이 액션 객체를 Redux에 보내 상태 변경을 요청합니다.

```js
const actions = {
  getSome: payload => ({ type: 'GET_SOME', payload }),
};
```

---

## Redux-Saga 기본

- Redux 비동기 작업 관리를 위한 라이브러리
- Generator 함수(`function*`)와 `yield`를 사용해 비동기 흐름 제어

```js
import { takeEvery, call, put } from 'redux-saga/effects';
import axios from 'axios';

function fetchUserApi(userId) {
  return axios.get(`/api/user/${userId}`);
}

function* fetchUser(action) {
  try {
    const response = yield call(fetchUserApi, action.payload.userId);
    yield put({ type: 'FETCH_USER_SUCCESS', payload: response.data });
  } catch (e) {
    yield put({ type: 'FETCH_USER_FAILURE', payload: e.message });
  }
}

function* watchFetchUser() {
  yield takeEvery('FETCH_USER_REQUEST', fetchUser);
}

export default watchFetchUser;
```

### 핵심 문법

- `function*` : Generator 함수 선언 (yield 사용 가능)
- `yield` : 함수 실행 일시 중지 및 제어 위임
- `put` : Redux 액션 dispatch 역할
- `call` : 비동기 함수 호출

---

## Generator 함수와 yield

- `yield`는 Generator 함수 안에서만 쓸 수 있음
- 일반 함수에서 `yield` 사용하면 문법 오류 발생

```js
function* gen() {
  yield 1;  // 정상
}

function normal() {
  yield 1;  // 오류
}
```

---

필요하면 추가 자료나 예제 더 만들어 드릴 수 있습니다.  
Git에 올려서 언제든 편하게 보세요! 😊
