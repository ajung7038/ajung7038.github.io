---
title: "[React] Redux란"
categories:
  - Spring
tags:
toc: true
toc_sticky: true
date: 2024-07-03 02:11:00 +0900
---

# ❗Redux는 JS 상태 관리 라이브러리이다.❗

## ✨ Redux 정의

Redux는 JS의 상태 관리 라이브러리로, react와 결합하여 많이 사용된다.

마치 프로젝트 안에서 사용하는 전역 변수처럼, 쿠키 저장, 자주 사용되는 변수 등을 쉽게 저장하고 값을 끌어다 쓸 수 있다.

그럼 여기서 말하는 `상태 관리 라이브러리`란 무엇일까?

### 상태 관리 라이브러리

상태 관리 라이브러리에 대해 알기 전, 리액트에서 말하는 '상태'에 대해 알 필요가 있다.

여기서 상태는 component 안에서 관리되는 것으로, 리액트는 단방향이므로 상태는 부모 컴포넌트를 통해 값을 주고 받는다.

즉, 자식 간 상태를 전달하려면 (자식 컴포넌트의 변수 값을 사용하거나 자식 컴포넌트를 호출하는 경우 등) 하나의 자식 컴포넌트가 부모 컴포넌트로 값을 보내고, 부모 컴포넌트가 또 다른 자식 컴포넌트에 값을 전달하게 된다.

그런데 주어야 할 값이 많아지면 어떻게 될까? 자식이 많아지면 어떻게 될까? 컴포넌트에서 컴포넌트로, 또 컴포넌트에서 컴포넌트로... 상태를 관리하는 상위 컴포넌트에서 계속 내려 받아야 하므로 코드를 한 번에 파악하기도 힘들어질 뿐더러, 성능 저하의 문제까지 발생할 수 있다. (Props Driling 이슈)

![image](https://github.com/2024-Java-Study/PassionatePro_Frontend/assets/80907516/ea497158-6382-42a9-9c34-866db2ec416a)

이러한 문제를 해결하기 위해 상태 관리 라이브러리가 필요한 것이다.

상태 관리 라이브러리는 전역 상태 저장소를 제공함으로써 컴포넌트에서 컴포넌트를 호출에 호출하는 것이 아닌, 전역 상태 저장소에서 손쉽게 꺼내 쓸 수 있다.

### Redux 세 가지 규칙

1. 하나의 애플리케이션 안에는 하나의 스토어가 있다.
2. 상태는 읽기 전용이다.
   <br /> 상태를 업데이트할 때는 setState, spread 연산자 등을 사용해 업데이트한다.
3. reducer은 순수한 함수여야 한다. == 똑같은 파라미터로 호출된 리듀서 함수는 <strong>언제나</strong> 똑같은 결과 값을 반환 해야 한다.

## ✨ Redux 사용법

### 1. 라이브러리 설치

- redux
- react-redux
- redux-actions

### 2. 액션 타입 & 생성 함수 정의

- src/store/modules/파일명.js

```js
// 액션 타입 정의
const 액션_이름 = "상대위치 (파일명/액션_이름)";

// 액션 생성 함수 정의
export const 함수명 = () => ({ type: 이름 });
```

### 2-1. 예시

```js
const CLICK_BUTTON = "button/CLICK_BUTTON";

// 파라미터가 있는 경우
export const clickButton = (isClick) => ({ type: CLICK_BUTTON, isClick });

// 파라미터가 없는 경우

export const clickButton = () => ({ type: CLICK_BUTTON });
```

액션 이름을 지을 때 문자열의 앞부분에 모듈 이름을 넣는다. 다른 액션들과 충돌을 미연에 방지하기 위함이다.

### 3. 초기 상태와 리듀서 정의

```js
// 초기 상태 정의
const 초기_상태_정의_변수 = {
  변수명: 값,
};

// 리듀서 작성
export default function 함수명(state = 초기_상태_정의_변수, action) {
  // 상태 변경 로직 작성
}
```

### 3-1. 예시

```js
// 초기 상태 정의
const init = {
  isChange: false,
  info: "초기화 값",
};

// 리듀서 작성
export default function clickButton(state = init, action) {
  switch (init) {
    case true:
      return {
        ...state,
        info: "버튼이 눌렸습니다.",
      };
    case false:
      return {
        ...state,
        info: "버튼이 눌리지 않았습니다.",
      };
    default:
      return state;
  }
}
```

나중에 스토어 만들 경우 리듀서를 필요로 하니, 리듀서 함수의 경우에는 꼭 export default 해 주기!

### 4. combineReducers

#### 1) modules 디렉토리에 index.js 파일 수정 (src/store/modules/index.js)

아래는 위에서 예시로 보였던 clickButton 예제이다. 실제로 프로젝트를 진행할 때는clickButton이 아닌 직접 만들었던 리듀서 관련 파일을 추가해 줘야 한다.

```js
import { combineReducers } from "redux";
import clickButton from "./clickButton";

export default combineReducers({
  clickButton,
  // 나머지 리듀서도 추가
});
```

### 5. 스토어 만들기

스토어는 프로젝트에 단 하나만 존재 해야 하기 때문에 스토어는 <strong>'src/index.js'</strong> 파일에 만들어 준다.

참고로, 스토어를 만들 때 createStore라는 함수를 사용하여 파라미터로 리듀서를 넣어 준다.

```js
// src/index.js

import React from "react";
import ReactDOM from "react-dom";
import { createStore } from "redux";
import rootReducer from "./store/modules";

import "./index.css";
import App from "./App";
import registerServiceWorker from "./registerServiceWorker";

const store = createStore(rootReducer);
console.log(store.getState());

ReactDOM.render(<APP />, document.getElementById("root"));
registerServiceWorker();
```

## ✨ 참고 자료

- https://hanamon.kr/redux%EB%9E%80-%EB%A6%AC%EB%8D%95%EC%8A%A4-%EC%83%81%ED%83%9C-%EA%B4%80%EB%A6%AC-%EB%9D%BC%EC%9D%B4%EB%B8%8C%EB%9F%AC%EB%A6%AC/
- https://hanamon.kr/%ec%83%81%ed%83%9c%ea%b4%80%eb%a6%ac%eb%8f%84%ea%b5%ac-%ed%95%84%ec%9a%94%ec%84%b1/
- https://velog.io/@velopert/Redux-3-%EB%A6%AC%EB%8D%95%EC%8A%A4%EB%A5%BC-%EB%A6%AC%EC%95%A1%ED%8A%B8%EC%99%80-%ED%95%A8%EA%BB%98-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-nvjltahf5e
