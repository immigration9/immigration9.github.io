---
layout: post
title: 'React Hooks'
date: 2019-03-04 16:00:00 +09:00
categories: 'react'
published: false
---

## State Hook: useState

`useState`: Hook. 로컬 상태 저장을 위해 사용된다. Rerendering 시에 React는 해당 상태 값을 보존한다.
현재 상태 값과 업데이트 하는 로직을 담은 한 쌍을 보존한다.

`this.setState`와 유사하긴 한데, 전과 새로운 상태를 합치지 않는다는 점에서 다르다.

`useState(argument)`안에 들어가는 argument는 초기값을 나타낸다. 0을 넣으면 0이 초기값이 된다.

```javascript
// 구체적인 사용법: 어떤 것이 들어가던 상관없다.
function HookComponentWithStates() {
  const [age, setAge] = useState(28);
  const [language, setLanguage] = useState('javascript');
  const [todolist, setTodoList] = useState([{ text: 'Go shopping' }]);
  const [classified, setClassification] = useState({ hello: 'world!' });
}
```

ES6 비구조화 할당을 사용하여, 선언된 상태 변수들에 각각의 다른 이름들을 줄 수 있다.
이 이름들은 `useState` API의 일부가 아니며, React는 `useState`가 여러번 호출될 경우, 매 렌더마다 같은 순서로 한다고 판단한다.

## Effect Hook: useEffect

side-effect 작업을 수행하기 위해 (데이터 가져오기, Subscribe, DOM 수동 조작 등) 사용된다.

`useEffect`는 function component 내에서 `componentDidMount`, `componentDidUpdate`, `componentWillUnmount`가 수행하던 항목들을 동일하기 수행하지만, 하나의 API로 통합된 형태로 제공된다.
