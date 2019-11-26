---
layout: post
title: 'React Hooks: Custom Hooks'
date: 2019-11-24 17:00:00 +09:00
categories: 'react'
published: true
---

## 직접 작성해보는 Custom Hooks

[React - Your First Custom Hook](https://itnext.io/react-custom-hooks-basics-f92de2a0ac0e)

Hook을 단순히 기존 class component를 functional component로 전환하는 것을 넘어, UI와 로직을 분리시킬 수 있다.

```javascript
function Form() {
  const [isValid, setValid] = useState(false);
  return (
    <div className="Form">
      <label htmlFor="password">Password: </label>
      <input
        id="password"
        onChange={e => {
          const newValue = e.target.value;
          let _isValid = false;
          if (newValue.length >= 8) _isValid = true;
          setValid(_isValid);
        }}
      />
      {isValid ? <p>Your password is valid </p> : null}
    </div>
  );
}
```

해당 UI에서 로직을 분리해보자.

```javascript
function Form() {
  const [isValid, onPasswordChange] = useSmartPassword();
  return (
    <div className="Form">
      <label htmlFor="password">Password: </label>
      <input id="password" onChange={e => onPasswordChange(e)} />
      {isValid ? <p>Your password is valid </p> : null}
    </div>
  );
}

function useSmartPassword() {
  const [isValid, setValid] = useState(false);

  const onChange = e => {
    /**
     * Longer validation logic can go here
     */
    const newValue = e.target.value;
    let _isValid = false;
    if (newValue.length >= 8) _isValid = true;
    setValid(_isValid);
  };
  return [isValid, onChange];
}
```

사실 이 로직이 다소 이해가 가지 않는 이유는, 다른 언어에 굉장히 흔한 tuple이 Javascript에는 없기 때문이다.

**일단 모든 Custom Hooks는 `currentValue`에 해당하는 값과, 해당 값을 업데이트하는 함수를 반환해야 한다.**
