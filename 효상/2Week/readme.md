# 2Week

## 1. 프로젝트 세팅

### 기본적인 폴더 구조

- public 폴더 : next 서버에서 접근 가능 (사이트에서 쓰일 이미지 등등 - 모든사람이 접근 가능)
- src : 원래 app이 src 밖에 있어야 하는데 한가지 예외로 가능 (src 폴더 내부에 mock , test 폴더 등등을 만들 수 있음)
- next.config.js : next에 대한 설정
- tsconfig.json : ts 설정인데 일단 기본 설정으로 사용
- @ : src 폴더를 뜻하며 절대 경로를 가질 수 있음
- npm run dev : 프로젝트 시작

## 2. 브라우저 주소 app 폴더에 반영하기

- RootLayout에 Page를 포함하므로 children을 리턴하고 있음
- 로그인을 했을 때 레이아웃과 로그아웃의 레이아웃이 서로 다르다.
- directory명을 [] 사이에 감싸면 그때그때 바뀐다
- NotFound : 404 NotFound 페이지 - 안걸리는 페이지는 다 여기로 연결
- 각각 폴더 만의 layout을 만들고 싶다면 각자의 layout을 만들어주면 된다.
- Rootlayout - Homelayout - Home(Page)의 계층을 만들 수 있다.

## 3. 라우트 그룹

- 로그아웃에 없어지려면 레이아웃을 어떻게 해야하는가,
- 로그인 후와 로그인 전만 분석을 해놓는다.
- 폴더에 (afterLogin) , (beforeLogin) 으로 나눈다. 해당 ()안에 있는 부분은 url을 안탄다.
- (afterLogin)을 적용 시키기 위해서는 얘만의 layout을 만들어 준다.

=> 처음에 폴더 구조를 잘 짜놔야 프로젝트 진행을 수월하게 한다.

## 4. template.tsx, Link, Image, redirect

- 리 렌더링이 안되고 싶으면 layout 되고 싶으면 template
- template은 공식 문서에 페이지 넘 나 들 때 기록이 필요할 경우에 필요함

=> login으로 갔다가 redirect로 보내려면 redirect 사용

####
import { redirect } from "next/navigation";

export default function Login() {
    redirect('i/flow/login');
}
####

=> Image는 Next가 Public 폴더 안에 있는 내용을 알아서 최적화 해줌

####
import Image from "next/image"
####

## 5. css module을 선택한 이유

- tailwind → 호불호가 심하고 가독성이 X
- Styled Component → SSR에서 문제가 좀 있음
- 간단하게 하기 위해 css module을 선택

=> global.css는 아예 모든 페이지에 적용하고 싶은 css를 넣는다.

=> 100dvw, 100dvh → 가장 쉽게 전체 화면을 채울 수 있는 기능 : 모바일에 주소창이 생기거나 사라짐과 상관없이 전체를 채운다.