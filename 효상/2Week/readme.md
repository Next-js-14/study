# 2Week

## 1. useSelectedLayoutSegement로 ActiveLink 만들기

### ActiveLink

- 다른 페이지로 이동이 가능한 사이드바(Link)를 클릭했을 때 현재 페이지가 active 상태로
- 현재 페이지가 아닌 페이지들은 active 하지 않은 상태를 가진 링크를 ActiveLink라 부른다.

### useSelectedLayoutSegement()

- Nextjs에서 제공하는 훅으로 클라이언트 컴포넌트에서 사용되는 훅
- 레이아웃보다 아래에 있는 active 경로를 읽을 수 있다.
- 따라서 활성된 하위 경로에 따라 스타일을 변경하는 상위 레이아웃 내부의 탭과 같은 탐색 UI에 유용하다.

## 2. 홈탭 만들면서 Context API 적용해보기

- Context : 정보통신 용어로서의 컨텍스트는 호출, 응답 간의 환경 정보 (문맥)

- 탭 마다 보이는 포스트가 다르기 때문에 어떤 탭인지에 대한 상태값을 가지고 있어야 함
- 따라서 추천 탭과 팔로우 중 탭은 클라이언트 컴포넌트라는 걸 유추해볼 수 있다.
- 현재 Tab의 상태를 전역으로 공유해야 하기 때문에 Tab에 대한 Provider로 정의하는 것.

### REACT 공식 문서

- context를 이용하면 단계마다 일일이 props를 넘겨주지 않고도 컴포넌트 트리 전체에 데이터를 제공할 수 있습니다.
- 일반적인 React 애플리케이션에서 데이터는 위에서 아래로 (즉, 부모로부터 자식에게) props를 통해 전달되지만
- 애플리케이션 안의 여러 컴포넌트들에 전해줘야 하는 props의 경우 (예를 들면 선호 로케일, UI 테마) 이 과정이 번거로울 수 있습니다.
- context를 이용하면, 트리 단계마다 명시적으로 props를 넘겨주지 않아도 많은 컴포넌트가 이러한 값을 공유하도록 할 수 있습니다.

####
class App extends React.Component {
  render() {
    return <Toolbar theme="dark" />;
  }
}

function Toolbar(props) {
  // Toolbar 컴포넌트는 불필요한 테마 prop를 받아서
  // ThemeButton에 전달해야 합니다.
  // 앱 안의 모든 버튼이 테마를 알아야 한다면
  // 이 정보를 일일이 넘기는 과정은 매우 곤혹스러울 수 있습니다.
  return (
    <div>
      <ThemedButton theme={props.theme} />
    </div>
  );
}

class ThemedButton extends React.Component {
  render() {
    return <Button theme={this.props.theme} />;
  }
}
####

####
// context를 사용하면 모든 컴포넌트를 일일이 통하지 않고도
// 원하는 값을 컴포넌트 트리 깊숙한 곳까지 보낼 수 있습니다.
// light를 기본값으로 하는 테마 context를 만들어 봅시다.
const ThemeContext = React.createContext('light');

class App extends React.Component {
  render() {
    // Provider를 이용해 하위 트리에 테마 값을 보내줍니다.
    // 아무리 깊숙히 있어도, 모든 컴포넌트가 이 값을 읽을 수 있습니다.
    // 아래 예시에서는 dark를 현재 선택된 테마 값으로 보내고 있습니다.
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// 이젠 중간에 있는 컴포넌트가 일일이 테마를 넘겨줄 필요가 없습니다.
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // 현재 선택된 테마 값을 읽기 위해 contextType을 지정합니다.
  // React는 가장 가까이 있는 테마 Provider를 찾아 그 값을 사용할 것입니다.
  // 이 예시에서 현재 선택된 테마는 dark입니다.
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
####

### 강의에서의 Provider 정의

####
"use client"; // 클라이언트 컴포넌트로 선언 (어디서든 사용 가능)

import { ReactNode, createContext, useState } from "react";

export const TabContext = createContext({
  tab: "rec",
  setTab: (value: "rec" | "fol") => {},
});

type Props = { children: ReactNode };

// children에 value를 모두 내려줌
export default function TabProvider({ children }: Props) {
  const [tab, setTab] = useState("rec");

  return (
    <TabContext.Provider value={{ tab, setTab }}>
      {children}
    </TabContext.Provider>
  );
}
####

## 3. classnames로 클래스 합성하기

- classnames 라이브러리를 사용하면 여러 클래스를 추가하거나, 조건부로 클래스명을 줄 수 있다.
- css modules를 사용할 때 상태에 따른 클래스명을 부여해야할 경우 유용하게 쓸 수 있다.