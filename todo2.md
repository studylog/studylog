TodoList의 전체적인 뷰를 생성하기 전에, babel.config.js를 설정했습니다.

---

투두리스트를 만들며 필요할 것 같은 항목들을 미리 alias로 만들어서 사용하기 쉽게 설정했습니다.

module-resolver는 왜 써야하는지 모르겠지만 안 쓰면 안 돌아가니깐 썼어여 .... 나중에 꼭 찾아보겠슴둥 ^^;;

module.exports = {
presets: ['module:metro-react-native-babel-preset'],

plugins: [
[
'module-resolver',
{
root: ['./src'],
alias: {
'~': './src',
'@assets': './src/assets',
'@components': './src/components',
'@pages': './src/pages',
},
},
],
],
};

---

한 페이지로 끝나는 간단한 프로젝트라 App.js에서 다 해결해도 되지만 공부하는 김에 페이지를 나누고 첫 페이지를 설정해보고 싶어서 src 하위에 Wrapper.js를 생성하여 연결했습니다.

index.js 에서 import 해주면 끝

import {AppRegistry} from 'react-native';
import Wrapper from './src/Wrapper';
import {name as appName} from './app.json';

AppRegistry.registerComponent(appName, () => Wrapper);

---

Wrapper.js에 기본적인 Todo List 틀을 잡았습니다.

TodoList를 입력하는 Insert 부분과 입력한 TodoList로 컴포넌트를 나누었고 styled-component를 사용하여 스타일링 했습니다.

import React from 'react';

import {Background, TitleWrapper, Title, Content} from './Wrapper.style';

import TodoInsert from '@components/TodoInsert';
import TodoList from '@components/TodoList';

const Wrapper = () => {
return (
<Background>
<TitleWrapper>

<Title>Jurr Todo List</Title>
</TitleWrapper>
<Content>
<TodoInsert />
<TodoList />
</Content>
</Background>
);
};

export default Wrapper;
