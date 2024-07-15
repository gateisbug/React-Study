# Nextjs Middleware: pathname
## Overall
Nextjs 공부를 하면서 서버 컴포넌트는 클라이언트 컴포넌트에 비해 여러 장점을 가지고 있는데, API를 수신할 때까지 렌더링을 유예하는 것이 가능하고 또는 API를 받아올때 렌더링을 해두지만 정보를 채우지 않는 것도 가능하다. SSR의 이점을 극단적으로 이용할 수도 있고.
### 하지만 dynamic routing에선?
dynamic routing을 사용하는 경우, usePathname이나 useParams를 이용해서 URL을 알아내야한다. 다만 이들은 `window` 객체를 사용하기 때문에 `"use client"`로 클라이언트 컴포넌트임을 명시해야한다. 이렇게 되면 서버 컴포넌트의 이점을 사용할 수 없게 된다. 따라서 다른 방법을 찾아야만한다.
### 그래서 middleware가 있다
middleware는 간단하게 말하면 페이지가 렌더링되기 전에 서버 측에서 실행하는 함수를 뜻한다. middleware에선 `Requeset` 객체와 `Response` 객체에 접근 할 수 있다.
## Usage
1. `middleware.ts`를 `app/` 디렉토리와 같은 레벨에 두어야 한다. 안 그럼 작동을 안 한다.
2. 미들웨어 파일을 만든다
```ts
import { NextRequest, NextResponse } from 'next/server'

export function middleware(request: NextRequest) {
  const requestHeaders = new Headers(request.headers);
  requestHeaders.set('x-pathname', request.nextUrl.pathname);
  requestHeaders.set('x-params', request.nextUrl.search);

  return NextResponse.next({
    request: {
      headers: requestHeaders,
    }
  });
}
```
3. page에서 사용한다
```tsx
import { headers } from 'next/headers';

export default function page() {
  const headersList = headers();
  const headerPathname = headersList.get('x-pathname') || "";
  const headerParams = headersList.get('x-params') || "";

  return (
    <div>
      <span>{headerPathname}</span>
      <span>{new URLSearchParams(headerParams).get('filter')}</span>
    </div>
  )
}
```
## 참고자료
- [문서보며 알아보는 nextjs 미들웨어](https://velog.io/@pds0309/nextjs-%EB%AF%B8%EB%93%A4%EC%9B%A8%EC%96%B4%EB%9E%80)
- [Nextjs Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware)
- [NextJS 서버 컴포넌트에서 url, pathname 접근하기](https://velog.io/@taeyooooon/NextJS-%EC%84%9C%EB%B2%84-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EC%97%90%EC%84%9C-url-pathname-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)

# Infinite Scroll
## Overall
- 리스트의 컨텐츠가 많은 경우 초기 렌더링에 시간이 길어지기 때문에 Lighthouse의 Perfomance 점수가 낮게 나올 수 밖에 없다.
## Code
- useInfiniteScroll.ts
```tsx
import { useEffect, useRef } from 'react'

export default function useInfiniteScroll(callback: () => void) {
  const loaderRef = useRef<HTMLDivElement | null>(null)

  useEffect(() => {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          callback()
        }
      },
      { threshold: 1 },
    )

    if (loaderRef.current) {
      observer.observe(loaderRef.current)
    }

    return () => {
      if (loaderRef.current) {
        // eslint-disable-next-line react-hooks/exhaustive-deps
        observer.unobserve(loaderRef.current)
      }
    }
  }, [callback])

  return loaderRef
}
```
- table.tsx
```tsx
'use client'

import { useEffect, useState } from 'react'

import { COLUMNS } from '@app/item/table/columns'
import { Box, Cell, Container, Row } from '@datum/item'
import { CircularProgress } from '@ui'
import useInfiniteScroll from '@util/useInfiniteScroll'

interface Props {
  data: ItemInterface[]
}

const LOADER = 10

export default function ItemTable({ data }: Props) {
  const [items, setItems] = useState<ItemInterface[]>([])
  const [visibleCount, setVisibleCount] = useState(LOADER)

  const loadMoreItems = () => {
    setVisibleCount((prev) => prev + LOADER)
  }
  const loader = useInfiniteScroll(loadMoreItems)

  useEffect(() => {
    setItems(data.slice(0, visibleCount))
  }, [visibleCount, data])

  const rowClickHandler = (item: ItemInterface) => {
    console.log(item)
  }

  return (
    <Container>
      <Box className='table-header'>
        <Row>
          {COLUMNS.map((v) => (
            <Cell
              key={v.value}
              data-type='th'
              data-key={v.value}
              className='fzp fwb fcs'
            >
              {v.label}
            </Cell>
          ))}
        </Row>
      </Box>

      <Box className='table-body'>
        {items.map((item) => (
          <Row
            key={`${item.name}_${item.index}`}
            data-type='row'
            onClick={() => {
              rowClickHandler(item)
            }}
          >
            {COLUMNS.map((v) => (
              <Cell
                key={`${v.label}_${item.index}_${v.value}`}
                data-type='td'
                data-key={v.value}
                className='fzp fwr fc'
              >
                {v.render ? v.render(item) : item[v.value]}
              </Cell>
            ))}
          </Row>
        ))}

        {data.length >= visibleCount && (
          <div
            style={{
              padding: '2rem',
              display: 'flex',
              justifyContent: 'center',
            }}
            ref={loader}
          >
            <CircularProgress />
          </div>
        )}
      </Box>
    </Container>
  )
}
```
## 참고자료
- [무한 스크롤 구현하기(Intersection Observer API)](https://leeseong010.tistory.com/145)