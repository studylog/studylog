# module-resolver란,

babel.config.js에서 alias를 설정하기 위해선 module-resolver 를 설치해야한다.

\*\*alias는 프로젝트를 진행하며 폴더에 쉽게 접근할 수 있도록 별칭을 추가하는 속성이다.

---

# module-resolver란,

ESLint를 사용한 Babel module-resolver로 이를 사용하면 전체 자바스크립트 응용 프로그램의 상대 가져오기를 정리 할 수 있다. (= 절대 경로를 설정할 수 있다는 뜻) 상대 경로가 있는 폴더를 위아래로 이동하지 않으려면 응용 프로그램의 중요한 경로에 별칭(alias)을 추가하여 이러한 영역에서 모듈을 쉽게 가져올 수 있도록 한다.

---

# module-resolver 설치

```javascript
yarn add -D babel-plugin-module-resolver

```

---

# babel.config.js 예시

```javascript
module.exports = {
  presets: ["module:metro-react-native-babel-preset"],

  plugins: [
    [
      "module-resolver",
      {
        root: ["."],
        alias: {
          "~": "./src",
          "@assets": "./src/assets",
          "@components": "./src/components",
          "@pages": "./src/pages",
        },
      },
    ],
  ],
};
```

---

# 설정을 변경한 후에는 캐시를 삭제해야 절대경로가 적용되는 것을 확인할 수 있습니다.

```javascript
yarn start --reset-cache

or

npm cache clean --force
```
