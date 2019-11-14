---
layout: post
title: 'React Hooks: Things to remind'
date: 2019-10-16 21:00:00 +09:00
categories: 'react'
published: true
---

## Reference

[React 공식문서: Hook의 규칙](https://ko.reactjs.org/docs/hooks-rules.html)

## 여러개의 Hook을 한 컴포넌트 안에서 사용할 경우

- 아래와 같이 Hook이 호출되는 순서에 의존한다.

```javascript
function Form() {
  // 1. name이라는 state 변수를 사용하세요.
  const [name, setName] = useState('Mary');

  // 2. Effect를 사용해 폼 데이터를 저장하세요.
  useEffect(function persistForm() {
    localStorage.setItem('formData', name);
  });

  // 3. surname이라는 state 변수를 사용하세요.
  const [surname, setSurname] = useState('Poppins');

  // 4. Effect를 사용해서 제목을 업데이트합니다.
  useEffect(function updateTitle() {
    document.title = name + ' ' + surname;
  });

  // ...
}
```

- 여기서 문제점이, 호출되는 순서에 의존하기 때문에, 조건문 등을 사용해서, 임의로 특정 Hook의 동작을 막아서는 안된다. (아래와 같은 코드는 Hook의 정상 작동을 방해한다.)

```javascript
{
  ...
  if (name !== '') {
    useEffect(function persistForm() {
      localStorage.setItem('formData', name);
    });
  }
  ...
}
```

## 자신만의 Hook 만들기

[todo]
