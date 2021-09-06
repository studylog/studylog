기본적인 환경설정은 되어있기 때문에 바로 프로젝트 생성

react-native init ReactNatieTodo

---

프로젝트 생성 후, 바로 실행시키려고 했더니 pod install 하라는 경고와 함께 실행이 되지 않아

오류에서 시키는대로 pod install 실행

cd ios
pod install

---

오류 내용과 똑같이 했지만, 실행이 되지 않아 레포 업데이트를 진행한 후, pod install 을 진행했다.

pod install --repo-update
pod install

---

다시 프로젝트 폴더로 돌아가서 프로젝트 실행

cd ..
react-native run-ios
