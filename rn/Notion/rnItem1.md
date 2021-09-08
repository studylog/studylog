# styled-components란,

자바스크립트 파일 내에서 css를 사용할 수 있게 해주는 css-in-js 라이브러리로 React 프레임워크를 주요 대상으로 한 라이브러리이다.

---

# styled-components 라이브러리 설치

```javascript
npm install --save styled-components

or

yarn add styled-components
```

---

기본 문법은 styled.[컴포넌트 이름] 뒤에 백틱(`)을 사용하여 작성한다. (tagged template literal 문법)

```javascript
export const 변수이름 = styled.컴포넌트 이름``;
```

---

적용

main.js

```javascript
import React from "react";

import { Wrapper } from "./TodoInsert.style";

const Main = () => {
  return <Wrapper></Wrapper>;
};

export default Main;
```

---

main.style.js

```javascript
import styled from "styled-components/native";

export const Wrapper = styled.View`
  flex: 1;
  flex-direction: row;
  justify-content: space-between;
  margin: 15px;
  padding: 5px;
  ${(props) =>
    props.black &&
    css`
      background-color: black;
    `};
`;

export const WrapperInner = styled(Wrapper)``;
```

---

# styled-components 장점

- 공통적인 스타일을 적용해야 하는 경우 중복되는 스타일을 재사용성 할 수 있다.

- 완성된 컴포넌트를 상속받아 이용 가능하다.

- 컴포넌트를 사용할 때 넣어준 props를 스타일링 할 때도 사용할 수 있다.

- ThemeProvider를 통해 전역 스타일을 제공한다.
