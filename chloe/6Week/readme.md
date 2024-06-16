### 백엔드 서버 세팅하기
- 필수적인 데이터베이스와 캐시 시스템을 설치해야 함. postgres, redis 설치하고 pgAdmin을 통해 데이터 베이스 서버를 설정함. 
- 설치가 완료되면 localhost:9090/api에서 swagger문서로 API 확인가능
- 주의할 점은 서버액션을 수행할 때 서버에서 실패가 발생하더라도 사용자에게 그 실패가 보이지 않는 경우가 있을 수 있음. 따라서 적절한 오류 처리가 필요함

### 로그인과 회원가입 실제로 하기
- 이미지 업로드 주소를 백엔드 주소로 바꾸기 위해 next.js의 rewrite라는 기능을 사용함. 이를 next-config.js에 적어주면 됨
```
module.exports = {
  async rewrites() {
    return [
      {
        source: '/upload',
        destination: 'http://backend-server-address/upload',
      },
    ];
  },
};
```
- 사용자가 로그인했는지 확인하는 기준은 세션 토큰 쿠키가 있느냐의 여부임. 프론트 서버 전용 세션 토큰 쿠키와 백엔드 라우터 토큰인 connect.sid를 사용함
- 백엔드 서버에서 로그인을 할 때, 서버는 connect.sid 토큰을 쿠키에 담아 프론트 서버로 전송함. 프론트 서버는 이를 받아서 사용자 브라우저에 쿠키를 심음
- 문자열로 된 토큰 쿠키를 객체로 변환하기 위해 cookie 라이브러리를 사용함
```
const cookie = require('cookie');
```
- 여러 브라우저가 프론트 서버를 통해 접근하기 때문에 프론트 서버에 로그인 쿠키를 심으면 개인정보 유출 문제가 발생할 수 있음. 따라서, 브라우저에 직접 쿠키를 심어야 함

### useMutation 사용하기
- useMutation은 비동기 작업의 상태를 관리하기 위해 사용됨.
- 이것은 isPending, isSuccess 등 상태관리를 통해 구현됨. 
- optimistic update: useMutation을 사용하면 반응속도를 빠르게 할 수 있음. 요청이 성공했다고 가정하고, 결과를 즉시 화면에 반영함. 만약 요청이 실패하면 원상태로 되돌림. 이는 매번 성공하는 상황에서는 매우 유용하지만, 에러 발생 상황에서는 위험할 수 있음. 
- 데브 툴을 통해 뮤테이션 동작 여부를 확인할 수 있음.

### optimistic update 적용하기
- Optimistuc update를 적용하면, 예를 들어 '팔로우'버튼을 클릭하면 즉시 '언팔로우'로 바뀌는 것과 같은 사용자 경험을 제공함
- 핵심 콜백 함수들:
  - onMutate: mutate가 호출될 때 실행됨
  - onError: 에러가 발생했을 때 롤백하는 코드를 작성함 
  - onSettled: 데이터가 최종적으로 업데이트된 후에 실행됨
```
const mutation = useMutation(
  newData => api.updateData(newData),
  {
    onMutate: async newData => {
      await queryClient.cancelQueries('dataKey');
      const previousData = queryClient.getQueryData('dataKey');
      queryClient.setQueryData('dataKey', old => ({
        ...old,
        ...newData,
      }));
      return { previousData };
    },
    onError: (err, newData, context) => {
      queryClient.setQueryData('dataKey', context.previousData);
    },
    onSettled: () => {
      queryClient.invalidateQueries('dataKey');
    },
  }
);
```