### 레이아웃 클론하기
- 애프터 로그인의 Layout.tsx 부터 작업
- 왼쪽/ 오른쪽영역에서, 왼쪽에서도 Flex-grow:1, 오른쪽에서도 Flex-grow:1 하면 알아서 가운데 정렬이 됨
- 부모랑 자식이랑 값이 같을 때는 자식을 inherit으로 하면 부모 width를 따라감

### 네비게이션 부분
- 활성화된 부분이 bold되며 주소와 연동해서 화면이 바뀌는 것: 액티브 링크
- 위치가 어디인지 알아야 하는데 클라이언트 컴포넌트여야 함.
- useSelectedLayoutSegment라는 훅을 쓰면 됨.
- EventListener 이런게 있으면 애초에 클라이언트 컴포넌트로 빼주는게 좋음
- 사용자 정보 부분은 나중에 백엔드에서 가져옴
- svg 복사할 때 아우터 html을 복사하면 됨
  
### 트렌드 키워드 부분
- 서버에서 트렌드를 불러와서 map 으로 반복문 처리를 하면 됨

### 홈탭 ContextAPI
- 탭별로 포스트 내용이 바뀌어야 하는데 여러가지 선택지가 가능: use context, react query도 가능
- 투명하게 처리해서 뒤쪽이 보이는 것은 백드롭 필터를 씀
- 여기서는 컨텍스트 API를 사용, 이 경우 프로바이더가 필요
- TabProvider.jsx를 생성해야 함
- TabProvider는 Tab과 Post의 공통 부모여야 함

### PostForm, Post 부분
- dayjs: 글이 몇 분전에 쓰여졌는지 표시하는 라이브러리(moment, date-fns도 동일)
- npm trends에 들어가서 라이브러리 비교해보기
- dayjs의 한글플러그인을 사용 : dayjs.locale(preset:'ko')
  
### ActionButton 부분
- npm i classnames : 조건부로 클래스를 합성해줘야 하는데 그 때 사용되는 cx라는 라이브러리
- 클래스네임을 합성해서 액션버튼에 불이들어오게 할수 있음
- clsx도 같은 기능을 하는 라이브러리임