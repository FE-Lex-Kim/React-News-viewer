# News-viewer README

<br>

## 트러블 슈팅

<br>

### useEffect를 사용했을때 그안에서 상태값이 바뀐경우 코드순서

<br>

useEffect를 사용했을때 그안에서 상태값이 바뀐경우 코드의 순서가 어떻게 바뀌는지 이해가 안됐다.

React 공식문서의 LifeCycle,

useEffect부분도 다시 정독한뒤

[Dan Abramov](https://rinae.dev/posts/a-complete-guide-to-useeffect-ko) 이 작성한 UseEffect 완벽 가이드 번역본을 읽고나니 정리가 되었다.

<br>

상태가 바뀌면 업데이트가 되고

클래스 컴포넌트에서는 렌더함수가 실행되는데 

함수 컴포넌트에서는 컴포넌트 함수자체가 다시 실행된다.

<br>

그 이후 상태가 변경된 변수가 useState()에 의해 retrun 값으로 변경이된다.

그이후 함수형 컴포넌트의 return의 상태의 변경부분만 비교한뒤 브라우저의 DOM에 적용한다.

<br>

useEffect를 사용하면 함수형 컴포넌트의 실행순서가 헷갈린다.

정확히 어떤식으로 Lifecycle이 동작하는지 알아야한다.

<br>

예제)

```jsx
import React, { useState, useEffect } from 'react';
import styled from 'styled-components';
import NewsItem from './NewsItem';
import axios from 'axios';

const NewsListBlock = styled.div`
  box-sizing: border-box;
  padding-bottom: 3rem;
  width: 768px;
  margin: 0 auto;
  margin-top: 2rem;
  @media screen and (max-width: 768px) {
    width: 100%;
    padding-left: 1rem;
    padding-right: 1rem;
  }
`;

const NewsList = () => {
  const [articles, setArticles] = useState(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    // async를 사용하는 함수 따로 선언
    const fetchData = async () => {
      try {
        console.log(1);
        setLoading(true);
        const response = await axios.get(
          'https://newsapi.org/v2/top-headlines?country=kr&apiKey=0a8c4202385d4ec1bb93b7e277b3c51f',
        );
        setArticles(response.data.articles);
        console.log(2);
      } catch (e) {
        console.log(3);
      }
      setLoading(false);
      console.log(4);
    };
    fetchData();
  }, []);

  // 대기 중일 때
  if (loading) {
    console.log(5);
    return <NewsListBlock>대기 중…</NewsListBlock>;
  }
  // 아직 articles 값이 설정되지 않았을 때
  if (!articles) {
    console.log(6);
    return null;
  }

  console.log(7);
  // articles 값이 유효할 때
  return (
    <NewsListBlock>
      {articles.map((article) => (
        <NewsItem key={article.url} article={article} />
      ))}
    </NewsListBlock>
  );
};

export default NewsList;
```

<br>

**코드실행순서**

1 . 컴포넌트가 마운트 되고난후

articles가 falsy값이므로 `6`이 콘솔에 찍힌다.

<br>

2 . 이후 마운트가 된이후 useEffect가 실행된다.

그이유는 useEffect의 두번째 인자로 `[]` 빈배열을 주었기 때문이다.

<br>

3 . useEffect 함수안에서 try문의 첫번째 `1` 이 찍힌후 setLoading으로 상태값을 변경했으므로 함수 컴포넌트가 reRendering되어 if문안의 loadting이 trusy값이므로 콘솔에 `5`가 찍힌후 대기중이 랜더가 된다.

<br>

4. 다시axios가 실행된후, setArticles가 실행되어 상태가 변하므로 함수컴포넌트가 리랜더링 된후 loading은 아직도 true값이므로 다시 콘솔에 `5` 가 찍한다.

<br>

5. 이후에 다시 useEffect로 돌아와 콘솔에 `2`가 찍힌다.

<br>

6. try문이 종료 된후 setLoading이 false가 되므로 다시 함수 컴포넌트가 리랜더링 되어 loading if문을 지나 articles if문을 지나 콘솔에 `7` 이 찍히고 retrun문으로 JSX가 반환된다.

<br>

7. 그이후 다시 useEffect로 돌아와 콘솔에 4가 찍히고 마무리 된다.