제가 생각하는 리액트의 가장 큰 장점은 컴포넌트를 나누고 컴포넌트를 자유롭게 재사용 할 수 있다는 것입니다.

---

이번 Todo List 에서도 컴포넌트를 나누고 이를 적용했습니다.

파일 구조는 크게 assets, components, pages 로 나누었습니다.

---

Wrapper 안에 내용을 입력하는 TodoInsert 와 내용을 출력하는 TodoList 로 나누었고,

리스트 안에 아이템은 TodoItem으로 컴포넌트화했습니다.

---

나눈 컴포넌트를 Wrapper.js에 적용했습니다.

```javascript
import React, { useState } from "react";

import { Background, TitleWrapper, Title, Content } from "./Wrapper.style";

import TodoInsert from "@components/TodoInsert";
import TodoList from "@components/TodoList";

const Wrapper = () => {
  return (
    <Background>
      <TitleWrapper>
        <Title>Todo List</Title>
      </TitleWrapper>
      <Content>
        <TodoInsert />
        <TodoList />
      </Content>
    </Background>
  );
};

export default Wrapper;
```
