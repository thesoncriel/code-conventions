# Coding Conventions: StyleSheet

## Selector Specificity
선택자 명시도는 아래와 같은 선택자 유형에 따라 점수가 매겨지고 우선순위가 정해지는 것을 의미 합니다.

다음을 먼저 참고하면 좋습니다.

  * [CSS Specificity](https://www.w3schools.com/css/css_specificity.asp)
  * [CSS 선택자 이해](http://www.nextree.co.kr/p8468/)

아래는 명시도 점수를 도표로 요약 한 것입니다.

| selector | score | note |
|:---------|:------|:-----|
| !important | 10000 | 약속된 유틸리티 외엔 **절대** 쓰지 않는다. |
| inline style | 1000 | 가급적 자제 한다. |
| #id | 100 | class name 으로 대체 한다. |
| .class | 10	| |
| tagname	| 1 | |
| universal | 0 | css 초기화 때 외에는 쓰지 않는다. |

주의해야 할 점은 위 표에서 보시다시피 **!important** 는 반드시 꼭 필요한 곳에서만 써야하고 가급적 쓰지 말아야 할 속성이라는 것입니다.
의도치 않게 마구 덮어씌워 side effect 를 발생시키는 주범이 될 수 있기 때문입니다.

아래는 선택자 조합 시 발생되는 명시도 점수 예시입니다.

```scss
* {} // 0점
div + p {} // 2점
.spec {} // 10점
#wrap > .spec {} // 110점
#wrap { background: red !important; } // ※주의: 무조건 덮어 씌웁니다!
```

## Selector Optimization
웹브라우저에서 웹문서를 CSS와 함께 렌더링 시, 선택자에 따라 렌더링 속도와 그에 따른 UX가 달라지기에 선택자 작성 시 항상 최적화 할 수 있는 방법을 이용합니다.

관련 내용은 아래 글을 참고 하세요.

  * [효율적인 CSS 작성하기](https://webclub.tistory.com/361)

요약하자면 다음과 같습니다.

  * Tag 및 Class 규칙을 이용할 것.
  * 자손(Descendant) 선택자 보다 자식(Child) 선택자를 사용 할 것.
  * descendant selector: div p
  * child selector: div > p
  * ID, class 및 tag 외 attribute 나 전체 (*) 선택자와 같은 universal selector 는 자제 한다.
      * 단 attribute selector 는 tag 및 class selector 와 연동하여 사용하는 것은 권장 된다.

가급적 class name 하나로 특정해서 사용하되 tag 를 이용할 시 child(>), adjacent sibling(+), general sibling(~) 등과 같이 특정지을 수 있도록 합니다.

```scss
// bad
.wrap {
  color: #333;
  div {
    color: #ccc;
  }
}
// good
.wrap {
  color: #333;
  & > div {
    color: #ccc;
  }
}
// best
.wrap {
  color: #333;
  & > .inner-wrap {
    color: #ccc;
  }
}
```

### Styled Components 사용 시

SC는 렌더링 되면 컴포넌트 태그 자체적으로 class 를 만들어서 적용됩니다. 따라서 template string 내부 스타일에 하위 태그, 혹은 클래스를 선택자로 가리킬 때는 아래와 같이 사용합니다.

```ts
// bad
const Wrap = styled.div`
  color: #333;
  div {
    color: #ccc;
  }
`;
// good
const Wrap = styled.div`
  color: #333;
  & > div {
    color: #ccc;
  }
`;
// best
const Wrap = styled.div`
  color: #333;
  & > .inner-wrap {
    color: #ccc;
  }
`;
```

## Nesting Depth

하위 요소에 대한 선택자를 구성 시 최대 3 depth를 초과하지 않도록 합니다.

가령 SCSS 에서 클래스명 작성 시 아래와 같이 중첩(nesting) 될 수 있습니다.

```scss
.nesting-wrap {
  background: #fff;
  & > .child-wrap {
    background: #f80;
    & > .under-wrap {
      background: #f00;
    }
  }
}
```

이 것은 아래와 같이 compile 될 것입니다.

```css
.nesting-wrap {
  background: #fff;
}
.nesting-wrap > .child-wrap {
  background: #f80;
}
.nesting-wrap > .child-wrap > .under-wrap {
  background: #f00;
}
```

이전에 강조했듯이, 자손(descendant) 선택자 대신 자식(child) 선택자를 쓰기를 권장합니다.

하지만 다음 예제와 같이 마크업 복잡도가 늘어나게 될 경우 Compile 된 CSS 코드량도 늘어나게 되므로 자손(child) 선택자가 불가피하게 필요한 경우가 발생될 수 있습니다.

```html
<div class="wrap">
  <div class="list">
    <div class="item">
      <h4>아이템 제목</h4>
      <div class="right-wrap">
        <figure>
          <a href="">
            <img src="" alt="이미지" class="image">
          </a>
          <figcaption>이미지 설명</figcaption>
        </figure>
      </div>
    </div>
  </div>
</div>
```

```scss
// 네스팅이 많아짐에 따라 코드가 많아짐
.wrap {
  & > .list {
    & > .item {
      & > .right-wrap {
        & > figure {
          & > a {
            & > .image {
              display: block;
            }
          }
        }
      }
    }
  }
}

// 좀더 간결하게..
.wrap {
  .image {
    display: block;
  }
}
```

애초에 저렇게 Depth 가 깊어지고 복잡해진다면 Refactoring 을 통해 컴포넌트 분할을 하는 것이 옳습니다.

아래는 React 일 경우의 예제입니다. (개략적으로 구분된 것이니 참고만 하세요!)

```tsx
import React, { FunctionComponent as FC } from 'react';
import styled from 'styled-components';

interface FigureProps {
  href: string;
  src: string;
  caption: string;
}

const FigureWrap = styled.figure`
  & > a {
    & > .image {
      display: block;
    }
  }
`;

const Figure: FC<FigureProps> = props => (
  <FigureWrap>
    <a href={props.href}>
      <img src={props.src} alt="이미지" className="image" />
    </a>
    <figcaption>{props.caption}</figcaption>
  </FigureWrap>
);

interface ItemProps extends FigureProps {
  title: string;
}

const ItemWrap = styled.div`
`;

const Item: FC<ItemProps> = props => (
  <ItemWrap>
    <div class="item-wrap">
      <h4>{props.title}</h4>
      <div class="right-wrap">
        <Figure {...props} />
      </div>
    </div>
  </ItemWrap>
);

interface ListProps {
  items: ItemProps[];
}

const ListWrap = styled.div`
`;

const List: FC<ListProps> = props => (
  <ListWrap>
    <div className="list">
    {props.items.map((item, idx) =>
      <Item key={idx} {...item} />
    )}
    </div>
  </ListWrap>
);
```

## Do Not This

아래와 같은 코딩은 하지 않도록 주의 합니다.

### 네스팅을 이용하여 클래스명을 나누기

SCSS 나 Styled Components 의 네스팅 기능을 이용하여 클래스명을 나누는 경우가 있는데, 이럴 경우 해당 클래스명을 코드에서 찾을 때 번거로워 질 수 있으므로 하지 말것을 권장합니다!

```scss
// wrong
.object {
  &-value {
    &-checking {
      // attrs..
    }
  }
  &-setting {
    &-disabled {
      // attrs..
    }
  }
}

// good
.object-value-checking {
  // attrs..
}
.object-setting-disabled {
  // attrs..
}
```

### 클래스명에 대시(-) 를 2번 이상 넣지 않기

보통 대시를 2번 이상 넣는 것은 다음과 같은 경우를 말합니다.

```scss
.object-value--disabled {
  // attrs..
}
.object-value {
  &--disabled {
    // attrs..
  }
}
```

대체로 BEM (Block Element Module) 작성 명칭을 이용한 것인데, 상태에 따라 필요한 클래스와 그 클래스명이 매우 길어질 수 있기 때문입니다.

또 한 상태(state)는 아래와 같이 따로 개별 클래스를 사용하는 것을 권장합니다.

```scss
.object-value {
  &.disabled {
    // attrs..
  }
}
```

## 규칙의 의의

- 선택자 명시도를 이해하고 스타일을 작성하면 발생되는 사이드 이펙트를 줄일 수 있습니다.
- 지나친 네스팅으로 인한 css 렌더링 효율 저하를 이해하여 보다 올바르게 코딩 할 수 있게 됩니다.
- 서로 다른 상태(state)에 대한 클래스명을 한가지로 통일하여 코드의 일관성을 높입니다.

## 읽어보기
### Cross Browsing

CSS3 는 Device 및 Web Browser 에 따라 지원여부가 다릅니다.

자주 확인되는 내용은 아래 링크를 참고 하면 업무에 도움이 될 것입니다. 🙂

  * [Flexible Box](https://caniuse.com/#feat=flexbox)
      * Styled Components 를 이용할 시 자동으로 vender prefix 를 적용 시켜줍니다.
  * [Filter Effects](https://caniuse.com/#search=filter)
      * IE 는 안된다 봐야 합니다.

### H/W Acceleration

HTML5 및 CSS3 이용 시 아래와 같은 속성을 사용하면 H/W 가속을 적용시켜 더 부드러운 렌더링 및 반응성을 이끌어낼 수 있습니다.

  * css3 의 translate, transform, perspective, filter, animation 속성 적용
  * video 또는 canvas 요소

더 자세한 내용은 아래 url 을 통해 확인 해 보세요!

  * [Naver D2 - 하드웨어 가속에 대한 이해와 적용](https://d2.naver.com/helloworld/2061385)
  * [CSS GPU 애니메이션 제대로하기](https://wit.nts-corp.com/2017/08/31/4861)

상기 블로그글을 요약 하자면 다음과 같습니다.

  * H/W 가속 렌더링 레이어 (이하 레이어)의 크기가 지나치게 크지 않도록 해야 합니다.
  * 필요한 곳에 알맞게 쓰지 않으면 의도치 않은 Memory Leak 이 발생되어 오히려 UX를 저하 시킬 수 있습니다.