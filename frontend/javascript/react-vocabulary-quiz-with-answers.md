# Vocabulary Quiz

In diesem kleinen Quiz versuchen wir das bewusstsein für eine Terminologie zu schärfen, damit sich unsere Entwickler auf einem hohen Niveau unterhalten können.

1. Lege in der folgenden _Klasse_ eine _Instanzvariable_ `state` als _Objekt_ an.

```js
class ColorPicker extends Component {
  state = {
  };
}
```

Diese Schreibweise gehört noch nicht offiziell zum EcmaScript Standard, allerdings ist davon auszugehen, dass das in Zukunft der Fall sein wird. Es gibt noch die Alternative den `state` im Konstruktor zu setzen. Diese Schreibweise ist dadurch etwas abwärtskompatibler (genauer ab ES6/2016), aber nicht empfehlenswert:

```js
class ColorPicker extends Component {
  constructor(props) {
    super(props);

    this.state = {
    };
  };
}
```

2. `state` soll ein _Objekt_ sein, in der zwei es zwei _Attribute_ `scrollLeft` und `toScrollLeft` gibt. Beide Werte sollen mit 0 _vorinitialisiert_ werden.

```js
class ColorPicker extends Component {
  state = {
    scrollLeft: 0,
    toScrollLeft: 0
  };
}
```

Protip: Synonyme für _Attribute_ eines Objects sind auch der englische Begriff _Property_ oder _Key_.

3. Lege in der folgenden _Klasse_ eine _statische Klassenvariable_ `propTypes` an. `propTypes` soll ein _Objekt_ sein.

```js
class ColorPicker extends Component {
  propTypes = {
  };
}
```

4. _Importiere_ das _Modul_ PropTypes

```js
import PropTypes from 'prop-types';

class ColorPicker extends Component {
  propTypes = {
  };
}
```

5. Gibt der Klassenvariable 'prop-types' ein Attribut `index`. Das Attribut soll eine Zahl und nicht optional sein.

```js
import PropTypes from 'prop-types';

class ColorPicker extends Component {
  propTypes = {
    index: PropTypes.number.isRequired
  };
}
```

6. Spendiere der _Klasse_ eine render _Funktion_

```js
import PropTypes from 'prop-types';

class ColorPicker extends Component {

  state = {
    scrollLeft: 0,
    toScrollLeft: 0
  };

  propTypes = {
    index: PropTypes.number.isRequired
  };

  render() {
  }
}
```

ProTip: In einer funktionalen Programmiersprache spricht man trotz des Objektorientierten Ansatzes von Klassen von _Funktionen_ anstatt von _Methoden_. Es kann trotzdem sein, dass der ein oder andere Entwickler in diesem Fall Funktionen trotzdem als Methoden bezeichnet.

7. Die Render Funktion soll ein `<div>` _zurückgeben_, in dem zunächst "Hello World" stehen soll

```js
import PropTypes from 'prop-types';

class ColorPicker extends Component {
  state = {
    scrollLeft: 0,
    toScrollLeft: 0
  };

  propTypes = {
    index: PropTypes.number.isRequired
  };

  render() {
    return (
      <div>
        Hello World
      </div>
    );
  }
}
```

8. Füge dem `<div>` eine _onClick Handler_ hinzu und lege den entsprechenden _Callback_ in der _Klasse_ an

```js
import PropTypes from 'prop-types';

class ColorPicker extends Component {
  state = {
    scrollLeft: 0,
    toScrollLeft: 0
  };

  propTypes = {
    index: PropTypes.number.isRequired
  };

  handleClick = (event) => {
  };

  render() {
    return (
      <div onClick={this.handleClick}>
        Hello World
      </div>
    );
  }
}
```

Es gibt noch mehrere Methoden den Callback zu definieren. Man kann den Callback auch als reguläre Funktion innerhalb der Klasse definieren, allerdings hat ist `this` in diesem Fall nicht definiert.

```js
handleClick() {
};
```

Um das Problem zu umgehen. Greifen die meisten Entwickler bei der Callback Definition zur folgenden Lösung:

```js
<div onClick={this.handleClick.bind(this)}>
```

Das hat aber den entscheidenen Nachteil, dass jedes mal wenn die `render` Funktion aufgerufen wird ein neuer Clickhandler generiert wird. Bei sehr häufigen Aufrufen führt das zu einer schlechten Performance. Generell gilt bei React: Eine Komponente sollte bei gleichen `props` und `state` stets dieselbe Ausgabe rendern. Zwar unterscheidet sich die Ausgabe des DOMs hier visuell nicht, aber der Clickhandler ist bei jedem `render` eine andere Funktion.

Nehmen wir unsere Lösung noch mal unter die Lupe:

```js
handleClick = (event) => {
};
```

Es handelt sich um eine Instanzvariable die einer Phat-Arrow-Funktion (mit automatischen binding an `this`) zugewieden wird.
