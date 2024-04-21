### flex container 안의 두 요소 가운데 정렬하기
- container에 `margin: 0 auto`
- container에 `justify-content: center`
- 각 flex 요소에 `flex-grow: 1`
- width: 100% 해도 잘 안먹는 경우가 있는데, 그 경우 `width: inherit;` 을 쓰면 100%가 먹는다.

### useSelectedLayoutSegment로 ActiveLink 만들기
- useSelectedLayoutSegment : 레이아웃보다 한 수준 아래에 있는 active 경로 세그먼트를 읽을 수 있다. // compose
- useSelectedLayoutSegment : 자식의 자식 이름까지도 받고 싶을 경우 // ['compose','tweet']

### PostForm 만들기(타이핑 외우기)
- Form의 경우 대부분 이벤트 리스너가 많기 때문에 클라이언트 컴포넌트라고 생각하면 좋다.

### classnames로 클래스 합성하기 (feat. npmtrends로 라이브러리 고르기)

- classnames 라이브러리를 사용하면 여러 클래스를 추가하거나, 조건부로 클래스명을 줄 수 있다.

import cx from "classnames";

- <div className={cx(style.commentButton, { [style.commented]: commented })}>
- <div className={cx(style.repostButton, reposted && style.reposted)}>
- <div className={cx([style.heartButton, liked && style.liked])}>