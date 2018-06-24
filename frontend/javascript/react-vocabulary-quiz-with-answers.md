# Vocabulary Quiz

In diesem kleinen Quiz versuchen wir das bewusstsein für eine Terminologie zu schärfen, damit sich unsere Entwickler auf einem hohen Niveau unterhalten können.

## Lege in der folgenden Klasse eine Instanzvariable `state` als Objekt an.

```js
class ColorPicker extends Component {
  state = {
  }
}
```

Es gibt noch die Alternative den `state` im Konstruktor zu setzen.

```js
class ColorPicker extends Component {
  constructor(props) {
    super(props);

    this.state = {
    }
  }
}
```

## `state` soll ein Objekt sein, in der zwei es zwei Attribute `scrollLeft` und `toScrollLeft` gibt. Beide Werte sollen mit 0 vorinitialisiert werden.

```js
class ColorPicker extends Component {
  state = {
    scrollLeft: 0,
    toScrollLeft: 0
  }
}
```

## Lege in der folgenden Klasse eine statische Klassenvariable `propTypes` an. `propTypes` soll ein Objekt sein.

```js
class ColorPicker extends Component {
  propTypes = {
  }
}
```

## Importiere das Modul PropTypes

import PropTypes from 'prop-types';

```js
class ColorPicker extends Component {
  propTypes = {
  }
}
```

## Gibt der Klassenvariable 'prop-types' ein Attribut `index`. Das Attribut soll eine Zahl und nicht optional sein.

import PropTypes from 'prop-types';

```js
class ColorPicker extends Component {
  propTypes = {
    index: PropTypes.number.isRequired
  }
}
```

## Spendiere der Klasse eine render Funktion

```js
class ColorPicker extends Component {
  state = {
    scrollLeft: 0,
    toScrollLeft: 0
  }

  propTypes = {
    index: PropTypes.number.isRequired
  }

  render() {
  }
}
```

ProTip: In einer funktionalen Programmiersprache spricht man trotz des Objektorientierten Ansatzes von Klassen von Funktionen anstatt von Methoden. Es kann trotzdem sein, dass der ein oder andere Entwickler in diesem Fall Funktionen trotzdem als Methoden bezeichnet.

## Die Render Funktion soll ein <div> zurückgeben, in dem zunächst "Hello World" stehen soll

```js
class ColorPicker extends Component {
  state = {
    scrollLeft: 0,
    toScrollLeft: 0
  }

  propTypes = {
    index: PropTypes.number.isRequired
  }

  render() {
    return (
      <div>
        Hello World
      </div>
    )
  }
}
```

## Füge dem <div> eine onClick Handler hinzu und lege den entsprechenden Callback in der Klasse an

```js
class ColorPicker extends Component {
  state = {
    scrollLeft: 0,
    toScrollLeft: 0
  }

  propTypes = {
    index: PropTypes.number.isRequired
  }

  handleClick = (event) => {
  }

  render() {
    return (
      <div onClick={this.handleClick}>
        Hello World
      </div>
    )
  }
}
```

