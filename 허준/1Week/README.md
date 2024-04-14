Weekly I Learned

[Next + React Query로 SNS 서비스 만들기] Section 0, Section 1

- 왜 Facebook(meta)은 React를 사용하게 되었는가?
  - 사용자가 유저 이름을 변경 했을때 기존의 구조로 업데이트를 할 수 없어서 React로 해결함
- Layout의 용도는?
  - Menu같이 고정해서 보여줄 경우 사용
  - Page 이동 시 Rendering이 안되는 경우
- Dynamic router
  - Folder name에 [ ]사용. 예) [username]/status/[post_id]
- Group("()")
- Template
  - Page 이동 시 Rendering이 필요한 경우 사용(새롭게 Mount 필요 할 때)
- Parallel Routing("@")
  - 여러 Page를 보여줄때. 예) Login Modal + Home Page
  - default.tsx(return null) = modal을 보여주지 않을때
- Client Component 사용하기
  - "use client";
- Intercepting Routing
  - 서로 주소가 다른데 같은 화면에서 보여줌
  - Link를 통해서 접근시 Intercepting 한다.
  - Browser에서 직접 주소 접근하거나 새로고침 했을때 Intercepting 하지 않는다.
- Private Folder("\_")
