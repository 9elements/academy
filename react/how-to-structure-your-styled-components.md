# How to structure your styled components

While developing your components you'll get to the point where you need to do integrate styles. Currently styled-components is the defacto standard in the React ecosystem to style your components.

The most ideal situation is that your design system is so developed that every design situation is covered by the component. Unfortunately while developing real world projects (under often budget and time constraints) this is not possible. You'll come to a point where you need to break out of your design system.

## Overriding styles with styled components

Overriding styles with styled components is actually pretty easy. If the component is already a styled component then you can use the `styled` function to add additional styles.

```js
const Card = styled.div`
  background-color: #e7e7e7;
  border: 1px solid #d7d7d7;
  border-radius: 1em;
`;

const CardWithMarginBottom = styled(Card)`
  margin-bottom: 1.5em;
`
```

If you component is a React component (either a class or a function component) you need to accept a `className` as property and pass this down to a styled component.

```jsx
const Background = styled.div`
  background-color: #e7e7e7;
  border: 1px solid #d7d7d7;
  border-radius: 1em;
`;

class Card extends Component {
  render(
    return
      <Background className={this.props.className}>
        {this.props.children}
      </Background>;
  )
}

const CardWithMarginBottom = styled(Card)`
  margin-bottom: 1.5em;
`;
```

Knowing that this is somewhat common accross your application you might want a quick way to structure and maybe even identify your styled components within your application. In the next sections I'll explain what are your options.

### Option 1: Prefix a styled component with Styled

This has been advised from a spectrum thread and I see this pattern quite often. In the most case a styled component has a `style.js` or `elements.js` that exports all styled components that are necessary for implementing styles on the component:

```js
export const StyledBackground = styled.div`
  background-color: #e7e7e7;
  border: 1px solid #d7d7d7;
  border-radius: 1em;
`;

export const StyledText = styled.h2`
  font-weight: bold;
`;

export const StyledText = styled.p`
  line-height: 1.5;
`;
```

the actual `Card` component would now import these styles.

```jsx
import {
  StyledBackground,
  StyledHeadline,
  StyledText
} from './style.jsx':

class Card extends Component {
  render(
    return
      <StyledBackground className={this.props.className}>
        <StyledHeadline>
          {this.props.headline}
        </StyledHeadline>
        <StyledText>
          {this.props.text}
        </StyledText>
      </StyledBackground>;
  )
}
```

The advantages of this approach are that all styles are accumulated in a single file that is almost readable by frontend people who are not used to React. One of the disadvantages is that this approach is pretty verbose and it might be annoying to type the prefix `Styled` all the time.

#### Prefix a styled component leveraging ES2016

To finetune the shortcomings. We give our components regular names:

```js
export const Background = styled.div`
  background-color: #e7e7e7;
  border: 1px solid #d7d7d7;
  border-radius: 1em;
`;

export const Text = styled.h2`
  font-weight: bold;
`;

export const Text = styled.p`
  line-height: 1.5;
`;
```

and then import them with a glob import:

```jsx
import * as S from './style.jsx':

class Card extends Component {
  render(
    return
      <S.Background className={this.props.className}>
        <S.Headline>
          {this.props.headline}
        </S.Headline>
        <S.Text>
          {this.props.text}
        </S.Text>
      </S.Background>;
  )
}
```

This helps to identify your Styled components very quickly.

#### Bonus: Give shared styles another prefix

At a certain point you'll repeat defining styles. Simply move them to a `shared/style.js` and give them a different namespace:

```jsx
import * as A from 'shared/style.jsx':
import * as S from './style.jsx':

class Card extends Component {
  render(
    return
      <A.PanelBackground className={this.props.className}>
        <S.Headline>
          {this.props.headline}
        </S.Headline>
        <S.Text>
          {this.props.text}
        </S.Text>
      </A.PanelBackground>;
  )
}
```

### Option 2: Following the BEM methodology with React (https://tech.decisiv.com/structuring-our-styled-components-part-i-2bf21fa64b28)

Since the first option works well with smaller and easy to style components it might not scale with larger applications that have visually complex components. If your `style.js` grows beyond 500 LOC you'll need something different.

The basic idea is to split up every styled component into a seperate file:

`BackgroundElement.js`:
```js
export default styled.div`
  background-color: #e7e7e7;
  border: 1px solid #d7d7d7;
  border-radius: 1em;
`;
```

`HeadlineElement.js`:
```js
export default styled.h2`
  font-weight: bold;
`;
```

`TextElement.js`:
```js
export default styled.p`
  line-height: 1.5;
`;
```

and then create a block from it:

```js
import BackgroundElement from './BackgroundElement';
import HeadlineElement from './HeadlineElement';
import TextElement from './TextElement';

class CardBlock extends Component {
  render(
    return
      <BackgroundElement className={this.props.className}>
        <HeadlineElement>
          {this.props.headline}
        </HeadlineElement>
        <TextElement>
          {this.props.text}
        </TextElement>
      </BackgroundElement>;
  )
}

CardBlock.BackgroundElement = BackgroundElement;
CardBlock.HeadlineElement = HeadlineElement;
CardBlock.TextElement = TextElement;
```

higher in your abstraction you would use a block like this:

```js
class Container extends Component {
  render(
    return
      <CardBlock.BackgroundElement className={this.props.className}>
        <CardBlock.HeadlineElement>
          {this.props.headline}
        </CardBlock.HeadlineElement>
        <CardBlock.TextElement>
          {this.props.text}
        </CardBlock.TextElement>
      </CardBlock.BackgroundElement>;
  )
}
```

While this approach is very flexible and it probably tickles the inner architect in you it also has some drawbacks.

- Exposing elements to components higher in the abstraction level might be a code smell. At it's core a component should be encapsulated with a minimal API surface (it's props) so that it can be easily replaced or rewritten. If you leak the elements structure higher in the abstraction tree then you tie them to your architecture and replacing or rewriting a component becomes more difficult.
