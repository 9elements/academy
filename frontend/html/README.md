# HTML

The HyperTextMarkupLanguage (HTML) is basically a text file with suffix ".html". So when you edit a file and just write:

```
Hello World!
```

and open this file in a web browser then the web browser will display the corresponding text. HTML origimates from back in the days when Tim Bernes Lee was thinking about a format to exchange documents. The idea was to enrich texts with markup to make them more semantic.

## HTML tags

So if your document has a headline you can simply write:

```
<h1>My first HTML page</h1>
Hello World!
```

If you open this page in your browser you'll see that the text `My first HTML page` is displayed in a different bolder font than the other text.

### Anatomy of a HTML tag

A HTML tag starts with the an angle bracket, a special character `<`, then comes the name of the tag in our case `h1` which stands for heading 1 and then the character `>`. This part is called the opening tag. For all the content that follows now the browser behaves differently until the closing tag occurs. A closing tag also starts with an angle bracket followed by a slash `</` and then again the name of the tag that you want to close, in our case `h1` followed by the closing angle bracket `>`.

