<h1 align="center">HTML Support in Typora</h1>

[toc]

The latest version of Typora already supports normal HTML tags.

Inline HTML tags, such as `<span>`, `<sup>` will be rendered right after you input their close tag, just as other Markdown syntax, like `**` or `__`. Other supported tags are rendered in a separate block and can switch between the output and HTML source code easily, just like math blocks.

For security convert, no scripts is supported, no matter you use `<script>` or `onload` attributes. `class`, `id`, and `data-*` are also not supported. For iframe, scripts are allowed inside `<iframe>`, but it will wrapped with `sandbox` attributes, and would have no access to your writing content nor local files.

You could find more details in following sections.

**Table of Contents**

- [Inline HTML](http://support.typora.io/HTML/#inline-html)
- [HTML Entities](http://support.typora.io/HTML/#html-entities)
- [HTML Block](http://support.typora.io/HTML/#html-block)
- Media and Embedded Contents
   - [Video](http://support.typora.io/HTML/#video)
   - [Audio](http://support.typora.io/HTML/#audio)
   - [Embed Web Contents](http://support.typora.io/HTML/#embed-web-contents)
   - [~~PDF~~](http://support.typora.io/HTML/#pdf)
- [Comments](http://support.typora.io/HTML/#comments)
- [`` or `**` ?](http://support.typora.io/HTML/#strong-or--)
- [Limitations](http://support.typora.io/HTML/#limitations)

## Inline HTML

Typora now can render inline HTML just as normal inline Markdown styles, for example:

| Raw Markdown Source                                          | Output in Live Preview |
| ------------------------------------------------------------ | ---------------------- |
| `<span style='color:red'>This is red</span>`                 | This is red            |
| `<ruby> 漢 <rt> ㄏㄢˋ </rt> </ruby>`                         | 漢ㄏㄢˋ                |
| `<kbd>Ctrl</kbd>+<kbd>F9</kbd>`                              | Ctrl+F9                |
| `<span style="font-size:2rem; background:yellow;">**Bigger**</span>` | **Bigger**             |
| `HTML entities like ® ¶`                                     | HTML entities like ® ¶ |

The writing experience is also same:

<video src="http://support.typora.io/media/html/inline%20HTML.mp4" style="width:100%;height:auto;" autoplay="" loop="" preload="" muted=""></video>

For easier editing purposes, Typora will show empty tags or HTML with `display:none` styles, for example, related contents in following Markdown will be visible in Typora, but invisible after export.

```
## <a name="anchor"></a> Header 2

<span style="display:none">I am hidden after export</span>
```

## HTML Entities

You could use [HTML entities](https://www.w3schools.com/html/html_entities.asp) directly in Typora, for example:

`¼` → ¼, `𝔗` → 𝔗

But we recommend to input their unicode directly, which is more readable and compatible.

## HTML Block

Block level HTML tags, and “invisible” tags (such as `script`, `meta`, etc) in Markdown document will be rendered as HTML Block, for example:

```
<details>
    <summary>I have keys but no locks. I have space but no room. You can enter but can't leave. What am I?</summary>
    A keyboard.
</details>
```

Will be rendered as:

<details style="display: block; color: rgb(0, 0, 0); font-family: Merriweather, &quot;PT Serif&quot;, Georgia, &quot;Times New Roman&quot;, STSong, serif; font-size: medium; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 300; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial;"><summary style="display: block;">I have keys but no locks. I have space but no room. You can enter but can't leave. What am I?</summary></details>

HTML Block can enter edit mode by when move cursor inside it, or click it non-interactive parts, or use `command`/`ctrl` + click.

Markdown syntax will not be parsed inside HTML blocks, which is the same for GFM/CommonMark.

For easier editing purposes, some inline tags, such as `svg` may also use the same editing behavior of those block level HTML tags .

Typora does not show preview for “invisible” tags, such as `<script>`, `<meta>` and `<style>`, but only shows their raw source.

## Media and Embedded Contents

### Video

You could embed a video like this:

```Markdown
<video src="xxx.mp4" />
```

Or drag & drop a video file into Typora, and Typora will insert the video automatically.

The path of `Video` follows the same rule of images. So, the option “Use relative path as possible”, and “image root path” also applies to `<video>` content.

### Audio

Same with `<video>`, you could use `<audio>` tag to embed an audio:

```
<audio src="xxx.mp3" />
```

<audio src="xxx.mp3" />

### Embed Web Contents

Some websites allow you to embed their contents into other webpages, most of them support `<iframe>`, which is also supported by Typora. You could just follow their “sharing” page/dialog, and paste their code into Typora, e.g:

```
<iframe height='265' scrolling='no' title='Fancy Animated SVG Menu' src='//codepen.io/jeangontijo/embed/OxVywj/?height=265&theme-id=0&default-tab=css,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/jeangontijo/pen/OxVywj/'>Fancy Animated SVG Menu</a> by Jean Gontijo (<a href='https://codepen.io/jeangontijo'>@jeangontijo</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
```

which will become

<iframe height="265" scrolling="no" title="Fancy Animated SVG Menu" src="http://codepen.io/jeangontijo/embed/OxVywj/?height=265&amp;theme-id=0&amp;default-tab=css,result&amp;embed-version=2" frameborder="no" allowtransparency="true" allowfullscreen="true" style="color: rgb(0, 0, 0); font-family: Merriweather, &quot;PT Serif&quot;, Georgia, &quot;Times New Roman&quot;, STSong, serif; font-size: medium; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 300; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-style: initial; text-decoration-color: initial; width: 800px;"></iframe>



Some websites only provide Javascript-based embed code, instead of an `<iframe>` snips, for example:

```
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Sunsets don&#39;t get much better than this one over <a href="https://twitter.com/GrandTetonNPS?ref_src=twsrc%5Etfw">@GrandTetonNPS</a>. <a href="https://twitter.com/hashtag/nature?src=hash&amp;ref_src=twsrc%5Etfw">#nature</a> <a href="https://twitter.com/hashtag/sunset?src=hash&amp;ref_src=twsrc%5Etfw">#sunset</a> <a href="http://t.co/YuKy2rcjyU">pic.twitter.com/YuKy2rcjyU</a></p>&mdash; US Department of the Interior (@Interior) <a href="https://twitter.com/Interior/status/463440424141459456?ref_src=twsrc%5Etfw">May 5, 2014</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
```

Typora only supports some script-based sharing code, and those contents/scripts will also be running in a sandbox iframe with no access to your local file and writing contents.

We could consider allowing users to config the “*whitelist*” for this kind in future updates.

### ~~PDF~~

No longer supported, you may try online file viewers instead, such as examples in https://gist.github.com/tzmartin/1cf85dc3d975f94cfddc04bc0dd399be.

## Comments

Typora supports HTML comments, using syntax `<!-- comments --> `, e.g:

```
<!-- I am some comments
not end, not end...
here the comment ends -->
```

They are invisible when export/print.

## `<Strong>` or `**` ?

Please use markdown syntax instead of original HTML tags, since the latter one are easier to input, and also better supported by Typora.

## Limitations

- In HTML Block, no empty line is not allowed, or it will be rendered as two HTML Blocks.
- Only common/normal HTML tags will be rendered as HTML contents in Typora, custom tags, like `<application>`, `<my-custom-component>` will be ignored (They will be included when export/print).
- Not all attributes are supported. `id`, `class`, `data-*` and unknown attributes in HTML will not be included when rendering (They will be included when export/print).
- Scripts are basically not allowed. `<style>` and `<meta>` won’t be applied either (They will be included when export/print).
- Not all HTML tags/styles can be exported to other formats. Export to PDF, HTML or HTML-compatible formats (such as EPub) would keep those HTML contents, but export to other formats, such as Word or LaTeX, those HTML contents may become plain text.