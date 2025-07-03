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
- state(상태) : 컴포넌트 내부에서 값을 저장하고 변경하는 공간
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
- API호출, 상태가 바뀔 때마다 어떤 작업을 하고 싶다면 사용하자

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
- Reducer: 상태 변경 함수(실제로 바꾸는 역할)

### React에서 Redux 사용하기
| Hook            | 역할                        |
| --------------- | ------------------------- |
| `useSelector()` | Redux store에서 **상태값 읽기**  |
| `useDispatch()` | **Action 보내기 (dispatch)** |


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

### React useState()만 사용할 때와 Redux를 사용할 때 차이
| 항목               | useState (로컬 상태)   | Redux (전역 상태)                   |
| ---------------- | ------------------ | ------------------------------- |
| **상태 저장 위치**     | 특정 컴포넌트 내부         | 전역 스토어 (store)                  |
| **다른 컴포넌트에서 접근** | ❌ 불가능 (props로만 전달) | ✅ useSelector로 접근 가능            |
| **복잡한 상태 흐름 제어** | ❌ 어려움              | ✅ 미들웨어로 제어 용이 (saga, thunk)     |
| **테스트 및 예측 가능성** | 🔸 컴포넌트마다 다름       | ✅ 순수 함수 기반으로 예측 가능              |
| **적합한 용도**       | 소규모 상태, UI용 상태     | 앱 전체에 걸친 상태, 사용자 정보, 토큰, 장바구니 등 |


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
- `yield` : 함수 실행을 중간에 멈췄다가 다시 시작할 수 있게 하는 Generator 함수 전용 키워드
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
