# Internet and Web Technology

The completed notes cover **Module I revision** and **JavaScript fundamentals, DOM and BOM**. Advanced JavaScript and Module III remain planned.

For the end-semester examination, give particular attention to page construction, form validation and XML/DTD design. These recur in the supplied papers. The full current syllabus remains the boundary; recurrence is evidence for study priority, not a prediction of the next paper.

| Start here | What to study |
| --- | --- |
| [Module I](#module-i---the-web-html-and-css-8-hours) | Web fundamentals, HTTP, HTML and CSS. |
| [Module II](#module-ii---javascript-dom-bom-and-advanced-concepts-10-hours) | JavaScript, DOM, forms and the browser environment. |
| [Module III outline](#module-iii---xml-ajax-jquery-json-reactjs-and-nodejs-12-hours) | Remaining prescribed topics and their question index. |

Actual examination questions appear after each broad topic. Marks are shown beside them; source notes retain the original filename, question number, page and any restriction on the retained portion. Original practice is labelled separately. Detailed evidence and scope decisions are kept in the appendices.

## Module I - The Web, HTML and CSS (8 hours)

This module is concise revision. Start with how the Web works, then connect HTML structure to CSS presentation.

### 1. Web Fundamentals and Architecture

The **Internet** connects computer networks. The **World Wide Web** uses that network to make linked pages and resources available through URLs and web protocols.

The distinction is:

> The Internet provides the network. The Web is one service that uses it.

Email can use the Internet without being a web page. A browser retrieves web resources and presents them to the user.

#### How the Web Developed

For the prescribed history, remember this short sequence:[^web-history][^internet-history]

| Year | Development |
| --- | --- |
| 1969 | ARPANET began. |
| 1983 | The adoption of TCP/IP helped establish the modern Internet. |
| 1989 | Tim Berners-Lee proposed the Web at CERN. |
| 1991 | The first website became publicly accessible. |

Hyperlinks made it easy to move between networked documents.

#### Websites and Web Projects

A **website** brings related pages together. A **web application** lets the user complete a task, such as registering or placing an order.

| Static website | Dynamic website |
| --- | --- |
| Serves stored pages without generating their contents for each request. | Can generate or update content using program logic and data. |

One site can combine information pages and application features.

A web project follows this sequence:

**Requirements → Content and navigation → Design → Implementation → Testing → Publishing → Maintenance**

For a tourism site, choose the audience and destinations first. Then organise the pages, implement the design and check the links, images and layout.

Important development issues include usability, accessibility, browser compatibility, loading time, input handling, security and maintainability.

#### Client-Server and Three-Tier Architecture

In the **client-server model**, the client requests a service and the server provides it.

The browser requests resources, interprets HTML/CSS and runs browser JavaScript. The web server accepts HTTP requests and returns resources or processing results.

A **three-tier architecture** separates presentation, application logic and data storage:

```text
Browser / presentation <--> Application / business logic <--> Database
          HTTP requests and responses       data operations
```

For registration, the browser shows the form, the application validates and processes the submission, and the database stores the accepted record.

These are logical responsibilities. They need not be three physical computers.

#### Protocols, Standards and MIME

A **protocol** defines communication rules. Standards allow browsers and servers from different suppliers to work together.

IETF publishes Internet protocol specifications. W3C develops many web standards, while WHATWG maintains the HTML Living Standard.

| TCP/IP layer | Main responsibility | Examples |
| --- | --- | --- |
| Application | Rules for a particular service. | HTTP for web resources; DNS for domain-name resolution. |
| Transport | Transfer between applications. | TCP gives reliable, ordered delivery; UDP does not give TCP's delivery guarantees. |
| Internet | Addressing and routing between networks. | IP. |
| Network access/link | Transfer over the local network. | Ethernet or Wi-Fi. |

In the HTTP/1.1 model, HTTP defines the exchange, TCP transports it reliably, and IP handles addressing and routing. HTTPS protects communication using TLS.

**MIME (Multipurpose Internet Mail Extensions)** identifies content types. A response's `Content-Type` can be `text/html`, `text/css`, `image/png` or `application/javascript`.

A MIME type describes the content. A filename extension is part of its name.

#### Cyber Laws

**Cyber laws** govern activity involving computers, networks and electronic communication.

They provide accountability for electronic transactions and misuse such as unauthorised access, identity theft and online fraud.

In India, the Information Technology Act, 2000 provides a central framework for electronic records/signatures and specified computer-related offences.[^cyber-law]

For this syllabus, know the purpose and examples. Detailed penalties and case law are outside the present treatment.

#### Examination Practice

**1. What stages and strategies are required to develop a web project?** (2 marks)[^pyq-01]

**Answer.** Identify the audience and requirements; plan content and navigation; design the interface; implement pages and functionality; test correctness, usability and browser compatibility; deploy and maintain the site. Use consistent layouts and reusable styles, and review the result against the requirements.

**2. Explain client-server, three-tier web architecture, and their difference.** (marks not shown)[^pyq-02]

**Answer.** In client-server computing, a client sends a request and a server processes it and returns a response. Three-tier web architecture divides the system into a presentation tier for interaction, an application tier for processing/business rules, and a data tier for persistent storage. For example, a browser submits a registration to an application, which stores it in a database and returns confirmation. Three-tier architecture organises client-server communication by separating presentation, application processing and storage, as shown in the diagram above.

---

### 2. HTTP: URLs, Messages and Connections

A browser needs to know **where a resource is** and **how to request it**.

A URL supplies the address. HTTP defines the request and response.

#### URL Anatomy

Consider:

`https://example.org:8443/books/list.html?topic=web#css`

| Part | Value | Meaning |
| --- | --- | --- |
| Scheme | `https` | How the resource is accessed. |
| Host | `example.org` | The server. |
| Port | `8443` | An explicit service port. |
| Path | `/books/list.html` | The resource on that server. |
| Query | `topic=web` | Query data. |
| Fragment | `css` | A location within the resource. |

The fragment is handled by the browser and is not sent in the HTTP request.

A relative URL such as `story.html` is resolved against the document's base URL.

#### HTTP Requests and Responses

**HTTP (Hypertext Transfer Protocol)** defines how a client requests a resource and how the server reports the result.

It is **stateless**: separate requests do not automatically become one remembered user session.[^http-overview]

A page may need several exchanges: first the HTML, then its stylesheets, images and scripts.

An HTTP/1.1 message contains:

1. A **start line**.
2. **Headers** describing the request or response.
3. A **blank line** ending the headers.
4. An optional **body** carrying content.

The request line gives the method, request target and version:

```http
GET /books/list.html?topic=web HTTP/1.1
Host: example.org:8443
Accept: text/html

```

The response begins with a version, status code and reason phrase:

```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 12

<h1>IWT</h1>
```

The illustrated body is 12 ASCII bytes without a trailing newline. `200` means success, `301` permanent redirection, `404` not found and `500` a server error.[^http-messages]

> HTTP defines the exchange. HTML is one type of content that it can carry.

#### GET and POST

| GET | POST |
| --- | --- |
| Requests a representation of a resource. | Submits data for processing. |
| An HTML GET form places data in the URL query. | Places submitted data in the request body. |
| Intended for retrieval without changing server state. | Commonly used to create or submit information. |

POST does not itself encrypt data. HTTPS supplies transport protection.

#### Connections, Caching and Proxies

A **non-persistent connection** carries one request-response exchange and then closes.

A **persistent connection** is reused for further exchanges, avoiding repeated setup. HTTP/1.1 uses persistence by default unless the connection is closed.

A **web cache** stores reusable responses. It can serve a fresh copy without downloading it again, or ask the server whether an older copy has changed.

Caching reduces delay, network traffic and server work. A stored copy is not permanently valid.

A **proxy** forwards exchanges between client and server. It may cache responses or apply access controls, but need not cache everything.

A browser cache is local to the browser; a proxy cache is at an intermediary.

> Persistence reuses a connection. Caching reuses a response. A proxy forwards the exchange.

#### Examination Practice

**1. Differentiate GET and POST. JSP tags are excluded.** (1 mark)[^pyq-03]

**Answer.** GET requests a representation of a resource; HTML GET forms put their data in the URL query. POST submits data in the request body for processing.

#### Additional Practice

**Original question · Suggested 4 marks.** Explain persistent connections and proxy caching, and state one benefit of each.

**Answer.** A persistent connection carries multiple HTTP exchanges over one TCP connection, reducing repeated setup overhead. A proxy cache stores reusable server responses between clients and origin servers. It can serve a fresh copy, reducing network traffic and response time; otherwise it forwards or revalidates the request. Persistence and caching can work together, but solve different problems.

---

### 3. HTML: Structure, Content and Forms

**HTML (HyperText Markup Language)** describes the structure and meaning of a page.

An element usually has an opening tag, content and a closing tag. Attributes add information about it. Void elements such as `img`, `meta` and `br` have no closing tag.

#### Document Structure

The `head` contains information about the page. The `body` contains the page content presented to the reader.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="description" content="IWT revision">
  <title>IWT study page</title>
  <base href="https://example.org/course/">
  <link rel="stylesheet" href="style.css">
  <style>h1 { color: navy; }</style>
  <script src="app.js" defer></script>
</head>
<body>
  <!-- This comment is not rendered as page text. -->
  <h1>Internet and Web Technology</h1>
  <p>Welcome to the course.</p>
</body>
</html>
```

The `doctype` selects standards mode. `title` names the browser tab, while `meta` supplies metadata.

`link` references an external resource, `style` contains CSS, and `script` includes JavaScript. The optional `base` changes relative-URL resolution: here, `style.css` resolves under the specified course URL.

A visible heading such as `h1` belongs to the body; it has a different role from `head`.

#### Text and Lists

Use `h1`-`h6` for heading levels, `p` for a paragraph and `br` for a line break within a passage. Use CSS for layout and spacing rather than repeated line breaks.

`strong` marks importance and `em` emphasis. `b` and `i` can mark text without those meanings. `sub`, `sup`, `mark`, `small`, `del` and `ins` provide common text formatting and annotations.

```html
<h1>Web revision</h1>
<p><strong>Remember:</strong> H<sub>2</sub>O and x<sup>2</sup>.</p>
<p><em>Read the example.</em><br>Then write it yourself.</p>
<ul><li>HTML</li><li>CSS</li></ul>
<ol><li>Plan</li><li>Implement</li><li>Test</li></ol>
<dl><dt>HTTP</dt><dd>A web request-response protocol.</dd></dl>
```

Choose the list according to the relationship between its items:

| List | Use |
| --- | --- |
| `ul` | Items whose order is not essential. |
| `ol` | A sequence of items. |
| `dl` with `dt` and `dd` | Terms paired with descriptions. |

#### Links, Images and Video

An anchor's `href` identifies its destination. An image's `src` identifies the image, and `alt` provides a text alternative.

```html
<a href="story.html">Read the travel story</a>
<a href="#contact">Go to contact details</a>
<img src="puri.jpg" alt="Beach at Puri" width="300" height="180">
<video controls width="320" poster="preview.jpg">
  <source src="tour.mp4" type="video/mp4">
  Your browser does not support this video.
</video>
<h2 id="contact">Contact details</h2>
```

The first link opens another page. The `#contact` link moves to the element with `id="contact"`.

Place an image inside an anchor to make it clickable. `width` and `height` specify dimensions. `alt` is an alternative to the image, not a tooltip; `title` may supply advisory text.

A `video` element can provide playback controls and alternative sources, as shown above.

#### Tables

A table presents data whose meaning depends on rows and columns.

`table` encloses it, `tr` creates a row, `th` marks a header cell and `td` a data cell. `caption` supplies a title.

`colspan` spans columns and `rowspan` spans rows. For example, `<tr><td colspan="2">Total</td></tr>` places one cell across two columns.

These attributes describe structure. CSS controls borders, padding, spacing and alignment.

#### Frames

An **iframe** embeds a separate document inside a page:

```html
<iframe src="story.html" title="Travel story"
        style="width: 100%; height: 180px; border: 1px solid black;"></iframe>
```

A DIV groups content in the same document; an iframe displays another document.

The syllabus also names historical frames. Recognise `<frameset cols="25%,75%"><frame src="menu.html"><frame src="main.html"></frameset>`: the frameset replaced the normal body in that older document form.

`frameset` and `frame` are obsolete in HTML5. Use an iframe for the retained layout question.[^frames]

#### Forms

A **form** collects related input. Its `action` identifies the receiving resource and its `method` determines the submission method.

Controls normally need a `name` to contribute submitted values. An `id` instead identifies an element in the document.

```html
<form action="/register" method="post">
  <label>Name <input name="student" required></label>
  <label>Course <select name="course"><option>IWT</option></select></label>
  <label><input type="checkbox" name="updates" value="yes">Send updates</label>
  <label>Message <textarea name="message"></textarea></label>
  <button type="submit">Register</button>
</form>
```

A label explains a control. Radio buttons with the same name represent alternatives; a checkbox represents an independent choice. Use `select` for options and `textarea` for multiline text.

A submit button initiates submission. `type="button"` performs only its programmed action.

HTML constraints and JavaScript can check input, but the receiving server must validate it again. Section 11 develops the complete validation process.

The larger travel, shopping, tourism and form-construction answers are grouped in Section 5.

#### Examination Practice

**1. Explain script-tag attributes. Detailed Java comparison is excluded; JavaScript characteristics are covered in [JavaScript Basics](#6-javascript-basics).** (1 mark)[^pyq-04]

**Answer.** `src` specifies an external script; `defer` downloads an external classic script without blocking parsing and executes it after parsing; `async` executes it when ready without preserving script order. `type` identifies the script type, for example `module`; ordinary JavaScript can omit it. Historical `language` is obsolete.

**2. Give image syntax and explain its attributes.** (2 marks)[^pyq-06]

**Answer.** `<img src="puri.jpg" alt="Beach at Puri" width="300" height="180">`. `src` is the image URL, `alt` is alternative text, and `width`/`height` set dimensions. It is a void element and needs no end tag.

**3. Explain anchor navigation and scrolling with an example.** (2 marks)[^pyq-07]

**Answer.** `<a href="story.html">Story</a>` opens another document. `<a href="#contact">Contact</a>` navigates to an element such as `<h2 id="contact">Contact</h2>` within the document. The fragment causes the browser to move to that target.

**4. Write HTML for the pictured Roll No./Student Name/Subject Name table.** (3 marks)[^pyq-09]

**Answer.**

```html
<style>
  table { border-collapse: collapse; }
  th, td { border: 1px solid black; padding: 4px; }
</style>
<table>
  <tr><th>Roll No.</th><th>Student Name</th><th>Subject Name</th></tr>
  <tr><td>1</td><td>Ram</td><td>Physics</td></tr>
  <tr><td>2</td><td>Shyam</td><td>Math</td></tr>
  <tr><td>3</td><td>Murli</td><td>Chemistry</td></tr>
</table>
```

**5. Explain tags/attributes for HTML/CSS tables with examples.** (2 marks)[^pyq-10]

**Answer.** `table`, `tr`, `th` and `td` define the table, rows, header and data cells. `colspan="2"` spans two columns and `rowspan="2"` two rows. `table { border-collapse: collapse; }` and `th, td { border: 1px solid; padding: 4px; }` style the table. The table in the preceding answer demonstrates the row and cell structure.

---

### 4. CSS: Styling and Page Layout

**CSS (Cascading Style Sheets)** controls presentation: colours, text, spacing and layout.

A rule such as `p { color: navy; }` has a selector (`p`) and a declaration (`color: navy`). The selector chooses elements; the declaration supplies a property and value.

#### Adding CSS and Selecting Elements

| Method | Example | Applies to |
| --- | --- | --- |
| Inline | `<p style="color: navy;">Hello</p>` | One element. |
| Internal | `<style>p { color: navy; }</style>` in `head` | The current document. |
| External | `<link rel="stylesheet" href="site.css">` | Documents linking to that stylesheet. |

An external stylesheet lets several pages share one design.

Selectors can describe an element, a group or a relationship:

| Selector | Meaning |
| --- | --- |
| `p`, `.note`, `#result`, `*` | Element, class, ID and universal selector. |
| `h1, h2` | Both sets of headings. |
| `article p`, `article > p` | Descendant paragraph; direct-child paragraph. |
| `h2 + p`, `h2 ~ p` | Next sibling paragraph; later sibling paragraphs with the same parent. |
| `input[type="email"]`, `[required]` | Matching attribute value; presence of an attribute. |
| `[href^="https"]`, `[href$=".pdf"]`, `[class~="note"]` | Starts with, ends with, or contains a space-separated word. |

A class can be shared; an ID should identify one element. Commas group selectors, while combinators express relationships.

`p.note` means a paragraph with that class. `.note p` means a paragraph inside an element with that class.

**Specificity** resolves competing selectors when their declarations are otherwise comparable. Compare IDs first, then classes/attributes/pseudo-classes, then elements/pseudo-elements. If priorities tie, the later rule wins.[^specificity]

```html
<style>
  p { color: blue; }
  .note { color: green; }
  #result { color: red; }
</style>
<p id="result" class="note">This text is red.</p>
```

The paragraph is red because `#result` has greater specificity.

Normal inline styling outranks normal stylesheet declarations. An applicable `!important` declaration can override a normal inline declaration. Therefore, placement alone does not decide every conflict.

#### Pseudo-Classes and Pseudo-Elements

A **pseudo-class** selects an element in a state or relationship, such as `a:hover`, `input:focus`, `a:visited` or `li:first-child`.

A **pseudo-element** selects part of an element or generates content, such as `::first-line`, `::first-letter`, `::selection`, `::before` or `::after`.

```css
a:hover { color: red; }
p::first-letter { font-size: 2em; }
p::first-line { font-weight: bold; }
.notice::before { content: "Note: "; }
.notice::after { content: " *"; }
::selection { background: gold; }
```

> A pseudo-class describes an element's state or relationship. A pseudo-element addresses part of it or generated content.

`::before` and `::after` normally need `content` to become visible. The double colon distinguishes pseudo-elements from pseudo-classes.

#### Colours, Backgrounds and Text Styling

`color` sets text colour. Colours may be named (`navy`), hexadecimal (`#003366`), RGB (`rgb(0,51,102)`) or include transparency (`rgba(0,51,102,0.5)`).

Background properties paint behind the content:

```css
body {
  color: #123;
  background-color: #eef6ff;
  background-image: url("paper.png");
  background-repeat: no-repeat;
  background-position: center top;
  background-size: cover;
  background-attachment: fixed;
}
/* Alternative shorthand for these background settings: */
.banner { background: #eef6ff url("paper.png") center top / cover no-repeat fixed; }
```

`repeat` tiles an image; `no-repeat` prevents tiling. `cover` fills the area and may crop the image. The usual `scroll` attachment moves with the element; `fixed` uses the viewport.

The shorthand can reset omitted background properties. An HTML image contributes content; a CSS background supplies presentation behind it.

Font, link, list and table rules make familiar structures readable:

```css
body { font-family: Arial, sans-serif; font-size: 16px; line-height: 1.5; }
h1 { font-weight: bold; font-style: normal; }
a:link { color: navy; }
a:visited { color: purple; }
a:hover, a:focus { text-decoration: underline; }
a:active { color: red; }
.icon { color: navy; font-size: 1.2em; }
ul { list-style-type: square; list-style-position: inside; }
table { width: 100%; border-collapse: collapse; }
th, td { border: 1px solid #777; padding: 6px; text-align: left; }
```

Use a generic fallback font. Link states are pseudo-classes.

An icon can be an image or a styled text symbol. For example, `<span class="icon" aria-hidden="true">&#9733;</span> Favourite` displays a star beside its label; no icon library is required.

`list-style-image: url("bullet.png")` supplies image bullets, while `list-style` combines list settings. Cell padding is inside a cell; `border-spacing` separates cells in a separated-border table. `border-collapse: collapse` merges adjacent borders.

#### The Box Model and Units

The **box model** places padding around content, a border around the padding, and margin outside the border:

```text
margin -> border -> padding -> content -> padding -> border -> margin
```

```css
.card {
  width: 200px;
  padding: 10px;
  border: 2px solid navy;
  margin: 15px;
}
```

With default `content-box` sizing, the border-box width is `200 + 20 + 4 = 224px`. Including horizontal margins gives `254px`.

$$
W_{\text{border box}} = W_{\text{content}} + P_{\text{left}} + P_{\text{right}} + B_{\text{left}} + B_{\text{right}}.
$$

With `box-sizing: border-box`, the declared width includes content, padding and border. **Margin stays outside the declared width in both models.** A visible border also needs a style such as `solid`.

| Unit | Relative to |
| --- | --- |
| `px` | A CSS pixel. |
| `%` | The relevant reference dimension. |
| `em` | Font size; the parent's font size when setting `font-size`. |
| `rem` | Root font size. |
| `vw`, `vh` | Percentages of viewport width and height. |

Four spacing values mean top/right/bottom/left. Two mean vertical/horizontal.

#### Display, Positioning and Alignment

`block` normally starts a new line, `inline` flows with text, and `inline-block` flows inline while accepting dimensions.

`display: none` removes the layout box. `visibility: hidden` preserves its space. For excess content, `overflow: auto` provides scrolling, `hidden` clips and `visible` allows overflow.

| Position setting | Effect |
| --- | --- |
| `static` | Normal flow. |
| `relative` | Offset while retaining the original space. |
| `absolute` | Removed from flow; usually placed against the nearest positioned ancestor's containing block. |
| `fixed` | Normally placed against the viewport. |
| `sticky` | Normal behaviour until a scrolling threshold is reached. |

`float: left/right` lets inline content wrap beside an element. `clear` prevents an element from sitting beside earlier floats.

Use `text-align: center/right` for inline content and `margin: auto` to centre a block with a suitable width. `vertical-align: middle` applies to inline-level or table-cell content, not arbitrary block boxes.

Multiple columns use `column-count`, `column-gap` and `column-rule`:

```css
.reading { column-count: 2; column-gap: 2rem; column-rule: 1px solid #aaa; }
.picture { float: left; margin-right: 1rem; }
.next { clear: both; }
```

#### Navigation, Galleries and Styled Forms

A navigation bar presents links; a dropdown reveals subordinate links; a gallery repeats image/link items. Form styles make labels and fields easier to scan.

```html
<style>
  nav > ul { list-style: none; padding: 0; }
  nav li { display: inline-block; position: relative; }
  nav a { display: block; padding: 8px; }
  .submenu { display: none; position: absolute; background: white;
             list-style: none; padding: 0; }
  li:hover > .submenu, li:focus-within > .submenu { display: block; }
  .gallery a { display: inline-block; margin: 8px; }
  .gallery img { width: 150px; border: 1px solid; }
  label { display: block; margin: 8px 0; }
  input { padding: 6px; border: 1px solid #777; }
</style>
<nav><ul><li><a href="#sites">Destinations</a><ul class="submenu">
  <li><a href="puri.html">Puri</a></li>
</ul></li></ul></nav>
<div class="gallery" id="sites">
  <a href="puri.html"><img src="puri.jpg" alt="Puri"></a>
</div>
```

The positioned parent provides a reference for the absolute submenu. The submenu is hidden until hover or focus reveals it. Gallery images use inline-block links.

#### Text Effects, Animations, Tooltips and Counters

**Opacity** affects the whole element, including its contents: 0 is transparent and 1 opaque. An alpha background colour affects only the background.

Text effects include shadows, wrapping and ellipsis. An animation changes properties through keyframes; a tooltip reveals a short explanation; a counter numbers repeated elements.

```html
<style>
  .faded { opacity: 0.5; }
  .heading { text-shadow: 1px 1px 2px #aaa; }
  .short { width: 12em; white-space: nowrap; overflow: hidden;
           text-overflow: ellipsis; }
  .wrap { overflow-wrap: anywhere; }
  @keyframes highlight { from { background: white; } to { background: gold; } }
  .notice { animation: highlight 2s infinite alternate; }
  .tip { position: relative; }
  .tip span { display: none; position: absolute; top: 100%; left: 0;
              padding: 4px; background: black; color: white; }
  .tip:hover span, .tip:focus span { display: block; }
  .steps { counter-reset: step; }
  .steps p::before { counter-increment: step; content: counter(step) ". "; }
</style>
<p class="faded">Half opacity</p>
<h2 class="heading">Revision</h2>
<p class="short">A long sentence that does not fit in this narrow area.</p>
<p class="wrap">averylongwordthatmustwrapinsideitscontainer</p>
<p class="notice">Exam notice</p>
<button class="tip" type="button">Help<span>Enter your student name.</span></button>
<div class="steps"><p>Read</p><p>Practise</p></div>
```

Ellipsis needs a constrained width and clipping. Animation needs named keyframes and a duration. Counters must be reset, incremented and displayed through generated content.

The tooltip appears on interaction. These are CSS mechanisms; the excluded historical `marquee` instruction is not needed.

#### Examination Practice

**1. Explain ways of using CSS in HTML, with examples.** (2 marks)[^pyq-15]

**Answer.** Inline CSS uses the `style` attribute, as in `<p style="color: navy;">Hello</p>`. Internal CSS uses a rule such as `<style>p { color: navy; }</style>` in `head`. External CSS uses `<link rel="stylesheet" href="site.css">`, with the rule in that stylesheet, allowing several pages to share it.

**2. Name the selector used to specify a group of elements and justify it.** (3 marks)[^pyq-16]

**Answer.** A **class selector** applies a shared style to a group of elements assigned the same class: `.notice { color: red; }` matches both `<p class="notice">Exam</p>` and `<h2 class="notice">Revision</h2>`. Unlike an ID, a class is reusable across elements. If “group” is used to mean a **grouping selector**, explain the comma form too: `h1, h2, p { color: red; }` applies one rule to several selector groups. The printed question does not distinguish these two meanings; state the interpretation rather than silently guessing.

**3. Define a pseudo-element and explain CSS pseudo-elements.** (2 marks)[^pyq-17]

**Answer.** A pseudo-element styles a selected part of an element. `::first-line` styles its first formatted line and `::first-letter` its first letter; `::before`/`::after` insert generated content; `::selection` styles selected text. For example, `p::first-letter { color: red; }`.

**4. Explain CSS background properties with examples.** (2 marks)[^pyq-18]

**Answer.** `background-color: lightblue` sets a color; `background-image: url("paper.png")` supplies an image; `background-repeat: no-repeat` prevents tiling; `background-position: center` positions it; `background-attachment: fixed` keeps it fixed during scrolling. These can be combined, for example `background: lightblue url("paper.png") no-repeat center fixed;`.

#### Additional Practice

**Original question · Suggested 3 marks.** A content-box element has width 180px, padding 8px, border 1px and margin 10px on every side. Find its border-box width and total horizontal space.

**Answer.** Border-box width = `180 + 2(8) + 2(1) = 198px`. Total horizontal space including margins = `198 + 2(10) = 218px`. Padding lies inside the border; margin lies outside it.

---

### 5. Complete Page-Design Questions

These questions combine HTML structure, navigation, CSS layout and simple page behaviour. Read the relevant concepts before writing the complete page.

**1. Display favourite holiday destinations as images linking to story pages; use external CSS for headings, images, links, margins and backgrounds.** (10 points)[^pyq-08]

**Answer.** The site consists of a destination index, linked story pages and one shared stylesheet. The named images are illustrative resources used by the answer.

`travel.html`:

```html
<!doctype html>
<html lang="en">
<head><meta charset="utf-8"><title>My travels</title>
<link rel="stylesheet" href="travel.css"></head>
<body>
<h1>My favourite destinations</h1>
<a href="puri.html"><img src="puri.jpg" alt="Puri beach">Puri</a>
<a href="konark.html"><img src="konark.jpg" alt="Konark temple">Konark</a>
</body></html>
```

`puri.html`:

```html
<!doctype html>
<html lang="en">
<head><meta charset="utf-8"><title>Puri story</title>
<link rel="stylesheet" href="travel.css"></head>
<body><h1>My visit to Puri</h1>
<p>I watched the sunrise and walked beside the sea.</p>
<a href="travel.html">Back to destinations</a></body></html>
```

`konark.html`:

```html
<!doctype html>
<html lang="en">
<head><meta charset="utf-8"><title>Konark story</title>
<link rel="stylesheet" href="travel.css"></head>
<body><h1>My visit to Konark</h1>
<p>I admired the carved wheels and learned about the Sun Temple.</p>
<a href="travel.html">Back to destinations</a></body></html>
```

`travel.css`:

```css
body { margin: 2rem; background: #eef6ff; font-family: sans-serif; }
h1 { color: navy; }
a { display: inline-block; margin: 1rem; color: #064b75; }
a:hover { color: #a33; }
img { display: block; width: 240px; height: 160px; border: 2px solid navy; }
```

The anchors make each destination image clickable; the shared stylesheet gives all pages consistent styling. Together, the pages provide the destination navigation and the corresponding travel stories.

**2. Retain the HTML registration form with first/last name, email, password, gender radio buttons and submit.** (retained portion; marks not separately given)[^pyq-12]

**Answer.**

```html
<form action="/register" method="post">
  <label>First name <input name="firstName" required></label>
  <label>Last name <input name="lastName" required></label>
  <label>Email <input type="email" name="email" required></label>
  <label>Password <input type="password" name="password" required></label>
  <fieldset><legend>Gender (optional)</legend>
    <label><input type="radio" name="gender" value="female">Female</label>
    <label><input type="radio" name="gender" value="male">Male</label>
    <label><input type="radio" name="gender" value="other">Other</label>
  </fieldset>
  <button type="submit">Submit</button>
</form>
```

The action is an illustrative endpoint, not an implementation. Servlet lifecycle, Java code, deployment descriptors and database-table creation from the original question are excluded.

**3. Retain the explicitly requested `index.html` login page.** (marks not shown)[^pyq-13]

**Answer.**

```html
<!doctype html>
<html lang="en"><head><meta charset="utf-8"><title>Login</title></head>
<body><form action="/login" method="post">
  <label>Username <input name="username" required></label>
  <label>Password <input type="password" name="password" required></label>
  <button type="submit">Log in</button>
</form></body></html>
```

This collects the credentials; authenticating them requires a server. The original servlet and `web.xml` portions are excluded.

**4. Design a local-product shopping site with catalogue, ordering and billing pages.** (5 marks)[^pyq-14]

**Answer.** Three connected pages present a local product, accept an order and display its bill. Here the product and price are illustrative choices, not details supplied by the paper. The GET form carries the quantity in the URL query; the billing page reads it and computes the displayed total. This is a front-end exam demonstration, not payment processing or authoritative server billing.

`catalogue.html`:

```html
<!doctype html>
<html lang="en"><head><meta charset="utf-8"><title>Local crafts</title>
<style>body { margin: 2rem; font-family: sans-serif; }
article { border: 1px solid; padding: 1rem; }</style></head>
<body><h1>Local crafts catalogue</h1><article>
<h2>Handwoven basket</h2><p>Price: Rs. 200 each</p>
<a href="order.html">Order this product</a>
</article></body></html>
```

`order.html`:

```html
<!doctype html>
<html lang="en"><head><meta charset="utf-8"><title>Order</title></head>
<body><h1>Order a basket</h1>
<form action="bill.html" method="get">
  <label>Quantity <input type="number" name="qty" min="1" max="20"
                         step="1" value="1" required></label>
  <button type="submit">Show bill</button>
</form><a href="catalogue.html">Back to catalogue</a></body></html>
```

`bill.html`:

```html
<!doctype html>
<html lang="en"><head><meta charset="utf-8"><title>Bill</title></head>
<body><h1>Your bill</h1><p id="bill"></p>
<a href="order.html">Change order</a>
<script>
  const params = new URLSearchParams(window.location.search);
  const qty = Number(params.get("qty"));
  document.getElementById("bill").textContent =
    Number.isInteger(qty) && qty >= 1 && qty <= 20
      ? "Basket: " + qty + " x Rs. 200 = Rs. " + (qty * 200)
      : "Enter a valid quantity on the ordering page.";
</script></body></html>
```

The catalogue leads to the order form, which passes the quantity to the billing page. Together they demonstrate the three requested pages, navigation, input checking and calculation.

**5. Design an HTML/CSS website about tourism sites of Odisha.** (5 marks)[^pyq-20]

**Answer.** The following single-page site provides a heading, destination information, navigation and CSS styling. The named images are illustrative resources for the page.

```html
<!doctype html>
<html lang="en"><head><meta charset="utf-8"><title>Explore Odisha</title>
<style>
  body { margin: 2rem; font-family: sans-serif; background: #eef6ff; }
  nav a { margin-right: 1rem; }
  article { padding: 1rem; margin: 1rem 0; border: 1px solid navy; }
  img { width: 240px; max-width: 100%; }
</style></head><body>
<header><h1>Explore Odisha</h1></header>
<nav><a href="#puri">Puri</a><a href="#konark">Konark</a></nav>
<article id="puri"><h2>Puri</h2>
<img src="puri.jpg" alt="Puri beach"><p>Visit its beach and temple town.</p></article>
<article id="konark"><h2>Konark</h2>
<img src="konark.jpg" alt="Konark Sun Temple"><p>Explore the Sun Temple.</p></article>
<footer>Plan your Odisha visit.</footer>
</body></html>
```

**6. Explain CSS usage and reproduce the pictured layout using the box model and different backgrounds. Retain float menu, centred header, right-aligned content/footer and inline-styled iframe; exclude the `marquee` instruction.** (marks not shown)[^pyq-21]

**Answer.** CSS controls presentation separately from HTML structure, making styles reusable. This layout uses a floated left column, an offset right column, and a cleared footer; borders/padding define the boxes.

```html
<!doctype html>
<html lang="en"><head><meta charset="utf-8"><title>CSS layout</title>
<style>
  * { box-sizing: border-box; }
  .page { width: 90%; margin: auto; border: 1px solid; }
  aside { float: left; width: 30%; background: #e6f2ff; }
  .menu { height: 200px; padding: 12px; border-bottom: 1px solid; }
  .right { margin-left: 30%; border-left: 1px solid; }
  header { height: 70px; text-align: center; background: #fff3cc;
           border-bottom: 1px solid; padding: 12px; }
  main { min-height: 300px; text-align: right; padding: 12px; background: #f2fff2; }
  footer { clear: both; border-top: 1px solid; padding: 12px;
           text-align: right; background: #eee; }
</style></head><body><div class="page">
<aside><div class="menu">Float Menu</div>
<iframe title="Notices" srcdoc="<p>Institute notices</p>"
        style="display: block; width: 100%; height: 170px; border: 0;"></iframe>
</aside>
<div class="right"><header>Header - Centre Align</header>
<main>Body content aligned right</main></div>
<footer>Footer - Right Align</footer>
</div></body></html>
```

---

## Module II - JavaScript, DOM, BOM and advanced concepts (10 hours)

Learn the language first. Then use it to work with the document, validate forms and interact with the browser.

### 6. JavaScript Basics

JavaScript makes a page respond to the user. It can calculate a bill, check a form or change a message after a button is clicked.

The relationship is:

> HTML gives the page structure. CSS controls its appearance. JavaScript supplies its behaviour.

#### Statements, Values and Variables

A **statement** performs an action. An **expression** produces a value.

In `const total = 3 * 200;`, multiplication produces a value and the declaration gives it a name. `//` begins a single-line comment; `/* ... */` encloses a block comment.

```javascript
const price = 200;          // This binding will not be reassigned.
let quantity = 2;           // It may change.
quantity = quantity + 1;
const total = price * quantity;
console.log(total);         // 600
```

`quantity` changes from 2 to 3, so `total` becomes 600.

JavaScript is **dynamically typed**: values have types, and a reassignable variable may later hold a different type.

Common values are numbers (`12`), strings (`"IWT"`) and booleans (`true`). `undefined` usually means an uninitialised or missing value; `null` can represent an intentional absence. Objects and arrays hold related data.

`12` and `"12"` are different: one is a number and the other is text.

#### `var`, `let` and `const`

**Scope** means where a variable can be accessed. A function creates a scope; a block groups statements inside braces.

| Declaration | Scope | Reassignment | Same-scope redeclaration | Before declaration is reached |
| --- | --- | --- | --- | --- |
| `var` | Function, or global at top level | Allowed | Usually allowed with another `var` | Reads as `undefined`. |
| `let` | Block | Allowed | Not allowed | Throws `ReferenceError`. |
| `const` | Block | Not allowed; initial value required | Not allowed | Throws `ReferenceError`. |

The difference becomes visible after leaving a block:

```javascript
function scopeExample() {
  if (true) {
    var x = 1;
    let y = 2;
    console.log(y);        // 2: inside its block
  }
  console.log(x);          // 1: still inside x's function
  // console.log(y);       // Would throw ReferenceError here.
}
scopeExample();
```

`x` belongs to the function, while `y` belongs to the inner block.[^bindings]

A `const` prevents reassignment, not changes inside an object. `const item = { qty: 1 }; item.qty = 2;` is allowed; assigning a different object to `item` is not.

#### Operators and Decisions

| Purpose | Operators |
| --- | --- |
| Arithmetic | `+`, `-`, `*`, `/`, `%` for remainder, `**` for powers. |
| Increment/decrement | `++`, `--`. |
| Assignment/update | `=`, `+=`, `-=`. |
| Comparison | `<`, `<=`, `>`, `>=`, `===`, `!==`. |
| Logical conditions | `&&`, `\|\|`, `!`. |

Strict equality avoids type conversion: `2 === "2"` is false, while `2 == "2"` is true. `typeof 12` gives `"number"`; `typeof "12"` gives `"string"`.

The `+` operator can also join text, so `"2" + 3` becomes `"23"`. Form values are strings: check for blank input, then use `Number(value)` before arithmetic. `Number("")` produces zero.

An `if/else` chooses a branch, the conditional operator expresses a short choice, and `switch` selects among cases:

```javascript
let marks = 65;
if (marks >= 40) {
  console.log("Pass");
} else {
  console.log("Fail");
}
const result = marks >= 40 ? "Pass" : "Fail";
switch (result) {
  case "Pass": console.log("Eligible"); break;
  default: console.log("Revise and retry");
}
```

Use parentheses when grouping might be unclear. These examples use ordinary small inputs; JavaScript numbers have finite precision.

#### Examination Practice

**1. Differentiate `let` and `var`.** (2 marks)[^pyq-22]

**Answer.** `let` is block-scoped, whereas `var` is function-scoped. `let` cannot be redeclared in the same scope; `var` generally can. A `var` binding can be read as `undefined` before its declaration is executed, while accessing `let` before initialisation throws an error. For example, in `if (true) { var a = 1; let b = 2; }`, `a` remains accessible in the enclosing function but `b` does not.

**2. Retain introductory JavaScript characteristics and script attributes; exclude detailed Java study.** (retained portion; original 1+1 marks)[^pyq-23]

**Answer.** JavaScript is dynamically typed and can run in the browser to handle events, validate forms and update the document. It is a separate language, not a variant of Java. Script inclusion/attributes are answered in [HTML: Structure, Content and Forms](#3-html-structure-content-and-forms). No separate retained-only mark total is inferred for the comparison.

---

### 7. Functions, Objects and Classes

A **function** packages an operation so it can be used with different inputs.

Parameters name the inputs inside the function. Arguments are the values supplied when calling it. `return` supplies the result and ends that call; without an explicit result, the function returns `undefined`.

```javascript
function cost(price, quantity) {
  const total = price * quantity;    // Local to this invocation
  return total;
}
console.log(cost(200, 3));           // 600
```

Here, `price` and `quantity` are parameters. `200` and `3` are arguments. The local `total` belongs to this call, and the caller receives 600.

#### Objects and Methods

An **object** groups named properties. A property containing a function is a **method**.

Use `student.name` for a known property, or `student[key]` when the name comes from a variable. Assignment adds or updates a property; `delete student.phone` removes one.

For an ordinary call `object.method()`, `this` refers to the receiving object:

```javascript
const student = {
  name: "Anu",
  marks: 65,
  result: function () {
    return this.name + ": " + (this.marks >= 40 ? "Pass" : "Fail");
  }
};
console.log(student.result());       // Anu: Pass
```

Because the call is `student.result()`, `this.name` and `this.marks` refer to that student's properties.

> The value of `this` depends on the call. It is not universally fixed by where the function was written.

Also distinguish `result` from `result()`: the first refers to a function, while the second calls it. This matters when registering event handlers later.

#### Classes and Instances

A **class** describes objects that share a structure and behaviour. `new` creates an instance, and `constructor` initialises its data.

```javascript
class Student {
  constructor(name, marks) {
    this.name = name;
    this.marks = marks;
  }
  result() { return this.marks >= 40 ? "Pass" : "Fail"; }
}
const anu = new Student("Anu", 65);
const ravi = new Student("Ravi", 30);
console.log(anu.result(), ravi.result()); // Pass Fail
```

The students share the same method, but each has its own name and marks. Changing one student's data does not change the other's.

#### Basic Inheritance

**Inheritance** lets a specialised class reuse a parent's behaviour. `extends` names the parent. In a subclass constructor, `super(...)` calls the parent's constructor before using `this`.

```javascript
class ResearchStudent extends Student {
  constructor(name, marks, topic) {
    super(name, marks);
    this.topic = topic;
  }
}
const scholar = new ResearchStudent("Mina", 70, "Web usability");
console.log(scholar.name, scholar.result()); // Mina Pass
```

The research student reuses the parent's name, marks and result, then adds a topic. A subclass may also replace an inherited method with its own implementation.

This establishes the prescribed class relationship; deeper advanced treatment remains planned.

#### Examination Practice

**1. State the purpose of `this` in JavaScript, with an example.** (2 marks)[^pyq-24]

**Answer.** In `object.method()`, `this` refers to the object on which the method is called, allowing access to its properties. For example, `const s = { name: "Anu", show: function () { return this.name; } }; console.log(s.show());` prints `Anu`.

**2. Add a function to an object and access it with an example.** (2 marks)[^pyq-25]

**Answer.**

```javascript
const item = { price: 200, qty: 3 };
item.total = function () { return this.price * this.qty; };
console.log(item.total());          // 600
```

`total` is a function-valued property; `item.total()` invokes it, with `this` referring to `item`.

#### Additional Practice

**Original question · Suggested 3 marks.** Define a Student class with a constructor and a pass/fail method, and show two objects with different results.

**Answer.** The Student class above supplies a constructor, properties, a result method and two instances. `this.marks` selects the current instance's marks: 65 produces `Pass`, while 30 produces `Fail`. The threshold is an illustrative rule for this generated question, not a rule stated in a past paper.

---

### 8. Strings, Numbers, Arrays and Dates

JavaScript provides built-in operations for the values that appear repeatedly in examination programs: text, numbers, arrays and dates.

The main question is:

> What kind of value am I working with, and what result should this operation return?

#### Strings

A **string** stores text. Character positions begin at zero, and `length` is a property: `"Web".length` is 3.

Strings are immutable. `" Web ".trim()` produces `"Web"` without changing the original value. `split(",")` separates text into an array, and `replace()` produces a replaced string.

The examination table below gives five further String methods and their results.

#### Numbers

`Number("12.5")` converts a whole numeric string. `parseInt("12", 10)` parses a decimal integer, while `parseFloat("12.5")` parses a floating-point value.

Check that a field is not blank before conversion. A conversion result alone does not prove that the user entered a number.

`Number.isNaN(value)` identifies the not-a-number value, and `Number.isFinite(value)` rejects infinity and NaN.

For display, `price.toFixed(2)` returns a string rounded to two decimal places. It does not create a new numeric type.

#### Arrays

An **array** is an ordered collection with zero-based indices. For `const a = ["HTML", "CSS"];`, `a[0] === "HTML"` and `a.length === 2`.

`push()` and `pop()` add and remove at the end. A `const` array can still have its contents changed.

Follow the array through these operations:

```javascript
const a = [10, 20, 30];
console.log(a.join("-"));           // "10-20-30"; array unchanged
const removed = a.splice(1, 1, 25); // Remove one at index 1; insert 25
console.log(removed, a);            // [20], [10, 25, 30]
console.log(a.shift(), a);          // 10, [25, 30]
console.log(a.unshift(5), a);       // 3, [5, 25, 30]
```

| Operation | Effect | Return value |
| --- | --- | --- |
| `join(separator)` | Joins elements without changing the array. | A string. |
| `splice(start, count, ...items)` | Removes/inserts elements in place. | Removed entries. |
| `shift()` | Removes the first entry. | The removed value. |
| `unshift(...items)` | Inserts at the front. | The new length. |

Do not confuse `splice()` with `slice()`, which copies a portion.

#### Dates and Mathematical Operations

`new Date()` obtains the current date/time value. `getFullYear()`, `getMonth()` and `getDate()` obtain local calendar components.

Months begin at zero. Add one for a human-readable month number. `getDay()` gives the weekday from 0 to 6, not the day of the month.

`toLocaleDateString()` gives a readable local date.

**Math** supplies numeric functions without constructing a Math object. The examples below include absolute value, square root and rounding.

`Math.random()` returns a value from 0 inclusive to 1 exclusive. For an integer from `low` to `high` inclusive, use:

`Math.floor(Math.random() * (high - low + 1)) + low`

Thus `Math.floor(Math.random() * 6) + 1` simulates a six-sided die.

#### Examination Practice

**1. Give five JavaScript Math methods and five String methods with examples.** (1+1 marks)[^pyq-26]

**Answer.** The following examples give five Math methods and five String methods with their results:

| Math example | Result | String example | Result |
| --- | --- | --- | --- |
| `Math.abs(-4)` | `4` | `"Web".charAt(0)` | `"W"` |
| `Math.sqrt(25)` | `5` | `"Web".indexOf("e")` | `1` |
| `Math.floor(2.8)` | `2` | `"Web".substring(1, 3)` | `"eb"` |
| `Math.ceil(2.2)` | `3` | `"Web".toUpperCase()` | `"WEB"` |
| `Math.round(2.6)` | `3` | `"Web".toLowerCase()` | `"web"` |

**2. Explain `join()`, `splice()`, `shift()` and `unshift()`.** (2 marks)[^pyq-27]

**Answer.** `join(separator)` returns a string of joined array elements; `splice(start, count, ...items)` removes/inserts elements in place and returns removed elements; `shift()` removes/returns the first element; `unshift(...items)` inserts at the front and returns the new length. The sequential example above shows all four results.

**3. Write a function to display today's date in the browser.** (2 marks)[^pyq-28]

**Answer.**

```html
<p id="today"></p>
<script>
  function showToday() {
    document.getElementById("today").textContent = new Date().toLocaleDateString();
  }
  showToday();
</script>
```

This uses the browser's current local date and display format. No particular date is hard-coded.

---

### 9. Loops, Regular Expressions and Errors

Many small examination programs combine three ideas: repeat a calculation, check that input has the right form, and handle problems clearly.

#### Loops

A **loop** repeats an operation while progressing towards an ending condition.

Identify its starting value, its condition and its update. A missing update may prevent termination; a wrong boundary may cause one extra or missing iteration.

| Loop | How it works |
| --- | --- |
| `for` | Keeps initialisation, condition and update together. |
| `while` | Before the body. |
| `do...while` | After the body, so it runs at least once. |

```javascript
let total = 0;
for (let i = 1; i <= 3; i++) total += i;
console.log(total);                 // 6
let n = 2;
while (n > 0) { console.log(n); n--; } // 2, 1
let count = 0;
do { count++; } while (count < 1); // Executes once.
for (const value of [10, 20]) console.log(value); // Values
for (const key in { name: "Anu", marks: 65 }) console.log(key); // Property keys
```

`for...of` visits array values. `for...in` enumerates property keys and is not the usual choice for array values.

`break` ends a loop; `continue` skips to the next iteration. Begin a sum at zero and a factorial product at one. Prime testing must reject values below 2.

> “The first n primes” counts how many to find. “Primes up to n” limits their values.

#### Regular Expressions

A **regular expression** describes a text pattern. It can check whether a mobile number contains only digits, for example.

A literal such as `/web/i` places the pattern between slashes. `i` ignores case; `g` requests multiple matches for operations such as replacement.

| Component | Meaning |
| --- | --- |
| `^`, `$` | Start and end of text. |
| `[0-9]` | One digit. |
| `+` | One or more repetitions. |
| `*`, `?` | Zero or more; zero or one. |
| `{n}` | Exactly n repetitions. |
| `.`, `\.` | Any character except a line terminator in ordinary mode; a literal full stop. |

`/[0-9]/` asks whether text contains a digit. `/^[0-9]+$/` requires the whole value to contain one or more digits.

```javascript
const digits = /^[0-9]+$/;
console.log(digits.test("0123"));     // true
console.log(digits.test("12a"));      // false
console.log("Web Technology".search(/technology/i)); // 4
console.log("web web".replace(/web/g, "IWT"));       // "IWT IWT"
```

`test()` belongs to a RegExp and returns a boolean. The string methods `search()` and `replace()` return the first match position (or `-1`) and a new string, respectively.

For repeated form checks, a plain anchored pattern without `g` avoids the retained `lastIndex` position used by global `test()` calls.

#### Errors and Exception Handling

| Error | What happens | Example |
| --- | --- | --- |
| Syntax | Code cannot be parsed. | A missing bracket. |
| Runtime | Failure during execution. | An undeclared variable is used. |
| Logical | Code runs but gives the wrong result. | Solving “primes up to n” instead of “the first n primes”. |

A `try...catch` statement handles an exception raised during an operation. `throw` deliberately reports an invalid condition; `finally`, when present, runs a final action after the try/catch.

```javascript
function checkedDivide(a, b) {
  if (b === 0) throw new Error("Divisor must not be zero");
  return a / b;
}
try {
  console.log(checkedDivide(10, 0));
} catch (error) {
  console.log(error.message);       // Divisor must not be zero
} finally {
  console.log("Calculation attempt finished");
}
```

The explicit divisor check is important: ordinary numeric division such as `10 / 0` produces infinity rather than automatically throwing.

A catch block cannot repair a script that failed to parse in the first place.

#### Debugging

**Debugging** means locating and correcting errors.

Reproduce the problem, read the console message and line number, then inspect relevant values. `console.log()` helps trace them.

Browser breakpoints or `debugger;` let you pause and step through execution. After correcting the problem, retest normal input and boundaries such as zero or empty input.

#### Examination Practice

**1. Find a number's factorial using input from a prompt dialog.** (2 marks)[^pyq-29]

**Answer.**

```javascript
const input = prompt("Enter a non-negative integer");
if (input !== null && input.trim() !== "") {
  const n = Number(input);
  if (!Number.isInteger(n) || n < 0) {
    alert("Enter a non-negative integer.");
  } else {
    let fact = 1;
    for (let i = 2; i <= n; i++) fact *= i;
    alert(n + "! = " + fact);
  }
}
```

For ordinary small exam inputs, `5` gives `120`; `0` gives `1` because the loop does not run. Prompt cancellation is distinguished from entering zero. Large-factorial arithmetic is not required by the question.

**2. Write `sum()` to find the sum of the first n prime integers, taking n from element `number` in form `f`.** (3 marks)[^pyq-30]

**Answer.** The function counts the primes it finds while testing candidates from 2 upward. A divisor greater than the square root is unnecessary because a composite has a factor no greater than its square root.

```html
<form name="f" onsubmit="sum(); return false;">
  <label>How many primes? <input name="number" type="number"
                                min="0" step="1" required></label>
  <button type="submit">Find sum</button>
</form><p id="primeResult"></p>
<script>
  function isPrime(value) {
    if (value < 2) return false;
    for (let d = 2; d * d <= value; d++) {
      if (value % d === 0) return false;
    }
    return true;
  }
  function sum() {
    const field = document.forms["f"].elements.namedItem("number");
    const n = Number(field.value);
    if (field.value.trim() === "" || !Number.isInteger(n) || n < 0) {
      document.getElementById("primeResult").textContent = "Enter a valid count.";
      return;
    }
    let found = 0, candidate = 2, total = 0;
    while (found < n) {
      if (isPrime(candidate)) { total += candidate; found++; }
      candidate++;
    }
    document.getElementById("primeResult").textContent = "Sum = " + total;
    return total;
  }
</script>
```

For `n = 5`, the selected numbers are 2, 3, 5, 7 and 11, so the result is 28. Do not stop at candidate 5: n is the count of primes, not the largest candidate.

**3. Define RegExp and explain `search()`, `replace()` and `test()` with examples.** (2 marks)[^pyq-31]

**Answer.** A regular expression specifies a text-matching pattern. `"IWT".search(/W/)` returns `1`; `"IWT".replace(/W/, "Web")` returns `"IWebT"`; `/^[0-9]+$/.test("123")` returns `true`. The three operations locate, replace and test a match, respectively.

**4. What is JavaScript debugging and how can it be done?** (2 marks)[^pyq-32]

**Answer.** Debugging is locating and correcting errors in JavaScript. Use the browser console to read errors and inspect values with `console.log()`. Use developer-tools breakpoints or `debugger` to pause and step through execution while watching variables, then correct and retest the code.

---

### 10. Working with the HTML DOM

The **Document Object Model (DOM)** represents an HTML document as a tree of objects.

JavaScript uses that tree to find elements, change content and styles, or add and remove nodes.

```text
Document
└── html
    ├── head
    │   └── title
    └── body
        ├── h1
        │   └── text: "IWT"
        └── p
            └── text: "Revision"
```

The HTML file describes the initial document. The live DOM can change after parsing, so a button can add a paragraph without rewriting the HTML file.

#### Finding Elements

`document` is the entry point. `document.documentElement` refers to `html`; `document.head`, `document.body`, `document.title` and `document.forms` give familiar access points.

| Method | Result |
| --- | --- |
| `getElementById("result")` | One element, or `null`. |
| `getElementsByTagName("p")` | A live HTMLCollection. |
| `getElementsByClassName("note")` | A live HTMLCollection. |
| `querySelector("p.note")` | The first CSS-selector match, or `null`. |
| `querySelectorAll("p.note")` | A static NodeList of all matches. |

Call these methods on `document`. A failed single selection returns `null`; a multiple selection may return an empty collection.

Run the script after the relevant elements exist, for example after their HTML or through an external script with `defer`.

#### Changing Content, Attributes and Styles

Use `.textContent` for plain text, `.innerHTML` for markup, and an input or textarea's `.value` for its current value.

Use `getAttribute()`, `setAttribute()` and `removeAttribute()` for attributes. For example, `image.setAttribute("alt", "Puri beach")` changes alternative text.

```html
<p id="message" class="note">Original text</p>
<script>
  const message = document.getElementById("message");
  message.textContent = "Revision started";
  message.style.backgroundColor = "lightyellow";
  message.setAttribute("title", "DOM example");
</script>
```

The example selects a paragraph, then changes its text, background and title attribute.

A CSS property such as `background-color` becomes `backgroundColor` through `.style`. Classes can be changed with `classList.add()`, `remove()` and `toggle()`.

`.style` reflects inline declarations. `getComputedStyle(element).color` reads the resolved style, including stylesheet rules.

#### Nodes and Navigation

A **node** can be a document, element, text or comment. Attributes can be accessed as attribute nodes, but are not ordinary child nodes.

| Node | `nodeType` |
| --- | --- |
| Element | `1` |
| Text | `3` |
| Comment | `8` |
| Document | `9` |

A text node's `nodeValue` holds its text. An element's `nodeValue` is `null`; use `.textContent` for its descendant text.

Navigation follows relationships such as `parentNode`, `childNodes`, `firstChild`, `lastChild`, `nextSibling` and `previousSibling`.

Whitespace can create text nodes. Therefore, `firstChild` may not be the first element.

For element-only navigation, use `parentElement`, `children`, `firstElementChild`, `lastElementChild`, `nextElementSibling` and `previousElementSibling`:

```html
<ul id="tasks"><li>Read</li><li>Practise</li></ul>
<script>
  const list = document.getElementById("tasks");
  const first = list.firstElementChild;
  console.log(first.textContent);           // Read
  console.log(first.nextElementSibling.textContent); // Practise
  console.log(first.parentElement === list);         // true
</script>
```

#### Adding and Removing Nodes

A newly created node is not visible until it is inserted into the document.

`createTextNode()` creates plain text. `appendChild()` inserts at the end; `insertBefore(newNode, referenceNode)` inserts before a child.

`replaceChild(newNode, oldNode)` replaces a child, and `removeChild(child)` removes one:

```javascript
// Continue with the UL from the example above.
const item = document.createElement("li");
item.appendChild(document.createTextNode("Revise"));
list.appendChild(item);
const extra = document.createElement("li");
extra.textContent = "Check examples";
list.insertBefore(extra, item);
const replacement = document.createElement("li");
replacement.textContent = "Write answers";
list.replaceChild(replacement, extra);
list.removeChild(item);
// Final element texts: Read, Practise, Write answers.
```

Appending an already attached node moves it rather than copying it.

A **NodeList** is an indexed collection, accessed with `[i]` or `item(i)`, measured by `.length` and traversed with a loop. It is not an Array and may include non-element nodes.

`querySelectorAll()` returns a static NodeList; `childNodes` returns a live NodeList. `children` and the `getElementsBy...` methods above return HTMLCollections.

```javascript
const snapshot = document.querySelectorAll("#tasks li");
const liveElements = list.children;
const before = snapshot.length;
const added = document.createElement("li");
added.textContent = "Review";
list.appendChild(added);
console.log(snapshot.length === before);     // true: snapshot membership unchanged
console.log(liveElements.length === before + 1); // true: live collection updated
```

> “Static” fixes the collection's membership. The nodes already in it still reflect edits to their contents.[^collections]

#### Events

An **event** reports an action such as a click, value change or form submission. A listener is the function that responds.

`addEventListener("click", handler)` registers a function. `handler()` would call it immediately. `onclick` provides one handler slot; `addEventListener` can register several listeners.

```html
<button id="change" type="button">Change message</button><p id="eventMessage"></p>
<script>
  document.getElementById("change").addEventListener("click", function (event) {
    document.getElementById("eventMessage").textContent = "Button activated";
    console.log(event.currentTarget.id); // change
  });
</script>
```

`click` reports activation, `input` reports value edits, and `change` reports a committed value change. `submit` reports form submission.

`keydown` and `keyup` respond to key actions; `load` reports resource loading. `DOMContentLoaded` reports that parsing has finished; deferred scripts can usually access elements without a separate readiness listener.

`mousedown` means a button was pressed and `mouseup` that it was released. `mouseover` and `mouseout` report entry and exit; movement across descendants can also trigger them because they bubble.

`event.target` is where an event originated. `event.currentTarget` is the element whose listener is running.

Many events reach ancestors after the target: this is **bubbling**. Event names in `addEventListener` omit the `on` prefix.

> `preventDefault()` prevents a default action, such as form navigation. `stopPropagation()` stops further propagation. They do different jobs.

#### Examination Practice

**1. Define HTML DOM; name three JavaScript methods for manipulating it and explain them with examples.** (5 marks)[^pyq-33]

**Answer.** The HTML DOM represents an HTML document as a tree of objects/nodes. JavaScript can use it to find elements, alter their content and add/remove nodes. Three useful methods are `getElementById()` to select an element, `createElement()` to create an element, and `appendChild()` to attach a node to a parent.

```html
<div id="content"></div>
<script>
  const container = document.getElementById("content"); // Select existing node.
  const paragraph = document.createElement("p");        // Create a detached p.
  paragraph.textContent = "Prepared using the DOM";     // Set its text.
  container.appendChild(paragraph);                    // Insert into the tree.
</script>
```

Initially the DIV is empty; afterwards it contains a paragraph. `createElement()` alone does not display anything: insertion into the document does. The example connects selection, creation and insertion in one operation.

**2. Access HTML elements through CSS selectors using JavaScript.** (2 marks)[^pyq-34]

**Answer.** `querySelector()` selects the first CSS-selector match; `querySelectorAll()` selects all matches.

```html
<p class="note">One</p><p class="note">Two</p>
<script>
  document.querySelector("p.note").textContent = "First note";
  const notes = document.querySelectorAll("p.note");
  for (const note of notes) note.style.color = "navy";
</script>
```

**3. Display a paragraph in a DIV with user-supplied width/height; wrap its text according to the DIV width.** (marks not shown)[^pyq-35]

**Answer.** Numeric inputs supply the dimensions, CSS controls wrapping, and the submit handler applies those dimensions and creates the paragraph.

```html
<form id="sizeForm">
  <label>Width in pixels <input name="width" type="number" min="1" required></label>
  <label>Height in pixels <input name="height" type="number" min="1" required></label>
  <label>Paragraph <textarea name="text" required></textarea></label>
  <button type="submit">Display paragraph</button>
</form>
<div id="textBox" style="border: 1px solid; overflow: auto;
                          overflow-wrap: anywhere; white-space: normal;"></div>
<script>
  document.getElementById("sizeForm").addEventListener("submit", function (event) {
    event.preventDefault();
    const fields = this.elements;
    const width = Number(fields.namedItem("width").value);
    const height = Number(fields.namedItem("height").value);
    if (!Number.isFinite(width) || !Number.isFinite(height) || width <= 0 || height <= 0) {
      alert("Enter positive dimensions."); return;
    }
    const box = document.getElementById("textBox");
    box.style.width = width + "px";
    box.style.height = height + "px";
    const paragraph = document.createElement("p");
    paragraph.textContent = fields.namedItem("text").value;
    box.textContent = "";
    box.appendChild(paragraph);
  });
</script>
```

Text normally wraps at spaces; `overflow-wrap: anywhere` also permits a long unbroken word to wrap. `overflow: auto` makes excess content accessible when the chosen height is too small. The dimensions need the CSS `px` unit when assigned as these strings.

**4. Write functions to handle `mouseup`, `mousedown`, `mouseover` and `mouseout`.** (2 marks)[^pyq-36]

**Answer.**

```html
<div id="zone" style="padding: 20px; border: 1px solid;">Use the mouse here</div>
<p id="mouseStatus"></p>
<script>
  const zone = document.getElementById("zone");
  const status = document.getElementById("mouseStatus");
  function up() { status.textContent = "Mouse button released"; }
  function down() { status.textContent = "Mouse button pressed"; }
  function over() { status.textContent = "Mouse entered"; }
  function out() { status.textContent = "Mouse left"; }
  zone.addEventListener("mouseup", up);
  zone.addEventListener("mousedown", down);
  zone.addEventListener("mouseover", over);
  zone.addEventListener("mouseout", out);
</script>
```

The four small functions and registrations directly answer the task. jQuery-specific event/effect methods in the uncertain 2023 question are not substituted for these plain JavaScript events.

---

### 11. Forms and Validation

A form combines the language and DOM ideas already studied.

The basic process is:

> Read the input → Check the rules → Display or submit the accepted result.

#### Reading Controls

Use `document.getElementById("name")` to obtain a control by ID, or `document.forms["f"].elements.namedItem("name")` to obtain it through a named form.

`.value` gives current input text. `.checked` tells whether a checkbox or radio button is selected.

Trim unwanted outside whitespace before checking text. Keep telephone numbers as strings to preserve leading zeros; convert values to numbers when doing arithmetic.

#### Validation

**Validation** checks whether input satisfies the required rules.

An empty name, letters in a numeric mobile field, or a quantity below one should be rejected before use. A regular expression is useful when the rule describes allowed characters.

HTML can declare many constraints directly:

| Constraint | Purpose |
| --- | --- |
| `required` | A value must be supplied. |
| `type="email"` | Email syntax. |
| `min`, `max`, `step` | Numeric range and step. |
| `minlength`, `maxlength` | Text length. |
| `pattern` | A text pattern. |

`checkValidity()` returns whether constraints pass. `reportValidity()` also displays the browser's validation message.

Use `setCustomValidity("message")` for a custom error and `setCustomValidity("")` to clear it after correction.[^validation]

#### Responding to Submission

A `submit` handler receives an event. `event.preventDefault()` stops the normal submission/navigation so that the page can show its result.

For plain output, use `.textContent`, or `.value` in a textarea. `innerHTML` would interpret the data as markup.

These examples demonstrate front-end behaviour. A real receiving server must validate again because browser checks can be bypassed.

#### Examination Practice

**1. Create a registration form for name, mobile and email; name must be nonempty/alphabetic, mobile numeric, email nonempty; show entered data in a textarea.** (marks not shown)[^pyq-37]

**Answer.** This version uses explicit JavaScript checks. The name check enforces letters only as stated; it does not add an unasked mobile length or email-domain rule.

```html
<form id="reg" novalidate>
  <label>Name <input id="regName" name="student"></label>
  <label>Mobile <input id="regMobile" name="mobile" inputmode="numeric"></label>
  <label>Email <input id="regEmail" name="email"></label>
  <button type="submit">Register</button>
</form>
<textarea id="regOutput" aria-label="Registration details" readonly></textarea>
<script>
  document.getElementById("reg").addEventListener("submit", function (event) {
    event.preventDefault();
    const name = document.getElementById("regName").value.trim();
    const mobile = document.getElementById("regMobile").value.trim();
    const email = document.getElementById("regEmail").value.trim();
    const output = document.getElementById("regOutput");
    output.value = "";
    if (!/^[A-Za-z]+$/.test(name)) {
      alert("Enter a nonempty alphabetic name."); return;
    }
    if (!/^[0-9]+$/.test(mobile)) {
      alert("Mobile must contain digits only."); return;
    }
    if (email === "") {
      alert("Email must not be empty."); return;
    }
    output.value = "Name: " + name + "\nMobile: " + mobile + "\nEmail: " + email;
  });
</script>
```

The anchored patterns test the whole name/number, not just one matching character. Output is cleared first so invalid resubmission cannot leave an earlier success visible.

**2. Design a vaccination-registration form, validate it using JavaScript, display submitted details and provide a confirm button.** (marks not verifiable)[^pyq-38]

**Answer.** The paper does not prescribe the fields, so name, age and contact below are illustrative form-design choices. The age check establishes a positive integer; it is not a vaccination-eligibility rule.

```html
<form id="vac">
  <label>Name <input id="vacName" required></label>
  <label>Age <input id="vacAge" type="number" min="1" step="1" required></label>
  <label>Contact <input id="vacPhone" pattern="[0-9]+" required></label>
  <button type="submit">Review details</button>
</form>
<pre id="vacDetails"></pre>
<button id="vacConfirm" type="button" disabled>Confirm</button>
<script>
  const vacForm = document.getElementById("vac");
  const vacDetails = document.getElementById("vacDetails");
  const vacConfirm = document.getElementById("vacConfirm");
  vacForm.addEventListener("input", function () {
    vacConfirm.disabled = true;
    vacDetails.textContent = "";
  });
  vacForm.addEventListener("submit", function (event) {
    event.preventDefault();
    if (!vacForm.reportValidity()) return;
    const name = document.getElementById("vacName").value.trim();
    if (name === "") { alert("Enter a name."); return; }
    vacDetails.textContent = "Name: " + name +
      "\nAge: " + document.getElementById("vacAge").value +
      "\nContact: " + document.getElementById("vacPhone").value;
    vacConfirm.disabled = false;
  });
  vacConfirm.addEventListener("click", function () {
    vacDetails.textContent += "\nDetails confirmed.";
    vacConfirm.disabled = true;
  });
</script>
```

The first action validates and displays; the second confirms the displayed data. Editing the form disables confirmation until the updated values are reviewed again.

**3. Design the HTML/CSS/JavaScript application page for IIIT Bhubaneswar's cyber-security hackathon: colleges enter groups of three; three winning groups receive prizes; students stay in institute residences and faculty mentors in the Guest House.** (5 marks)[^pyq-39]

**Answer.** The page presents the event conditions, collects an institute, exactly three participants and their mentor, and uses JavaScript to show the application. The paper does not ask for a server/database.

```html
<!doctype html>
<html lang="en"><head><meta charset="utf-8"><title>Hackathon application</title>
<style>
  body { font-family: sans-serif; max-width: 650px; margin: 2rem auto; }
  form { padding: 1rem; border: 1px solid navy; background: #eef6ff; }
  label { display: block; margin: 8px 0; }
  input { padding: 5px; }
</style></head><body>
<h1>IIIT Bhubaneswar Cyber Security Hackathon</h1>
<p>Technical colleges of Odisha may apply in teams of three.
Prizes will be awarded to the three winning groups. Students will stay in
institute residences; each college's faculty mentor will stay in the Guest House.</p>
<form id="hack">
  <label>College <input name="college" required></label>
  <label>Participant 1 <input name="p1" required></label>
  <label>Participant 2 <input name="p2" required></label>
  <label>Participant 3 <input name="p3" required></label>
  <label>Faculty mentor <input name="mentor" required></label>
  <label>Contact email <input name="email" type="email" required></label>
  <button type="submit">Review application</button>
</form><pre id="hackResult"></pre>
<script>
  document.getElementById("hack").addEventListener("submit", function (event) {
    event.preventDefault();
    const fields = this.elements;
    const names = ["college", "p1", "p2", "p3", "mentor", "email"];
    document.getElementById("hackResult").textContent = "";
    for (const name of names) {
      if (fields.namedItem(name).value.trim() === "") {
        alert("Complete every field."); return;
      }
    }
    document.getElementById("hackResult").textContent =
      "College: " + fields.namedItem("college").value +
      "\nTeam: " + fields.namedItem("p1").value + ", " +
      fields.namedItem("p2").value + ", " + fields.namedItem("p3").value +
      "\nMentor: " + fields.namedItem("mentor").value +
      "\nContact: " + fields.namedItem("email").value +
      "\nStudents: institute residences. Mentor: Institute Guest House.";
  });
</script></body></html>
```

The three required participant controls match the team-size rule; the native required/email checks run before a valid submit event, and the handler rejects whitespace-only entries. The page combines the event information, required controls, styling and a JavaScript review action.

**4. What is constraint validation, and which attributes can achieve it?** (2 marks)[^pyq-40]

**Answer.** Constraint validation checks whether form input satisfies declared restrictions. For example, `<input type="number" min="1" max="10" required>` accepts a required number within the range; `pattern`, `minlength`, `maxlength` and suitable input types provide other restrictions. JavaScript can test them using `checkValidity()` or show messages with `reportValidity()`.

---

### 12. The Browser Object Model

The **Browser Object Model (BOM)** refers to facilities provided by the surrounding browser environment.

The distinction is:

> DOM concerns the document. BOM concerns the window, history, browser information and timers.

`window` is the browser window/global object, and `window.document` refers to its document. BOM is a conventional teaching grouping rather than one single interface specification.

#### Window and History

| Facility | Purpose |
| --- | --- |
| `window.alert("Saved")` | Displays a message. |
| `prompt("Name?")` | Returns entered text or `null` if cancelled. |
| `confirm("Continue?")` | Returns a boolean. |
| `window.innerWidth`, `window.innerHeight` | Viewport dimensions in CSS pixels. |
| `outerWidth`, `outerHeight` | Dimensions including the outer browser window. |
| `history.back()`, `history.forward()`, `history.go(-1)` | Session-history navigation. |
| `history.length` | Number of entries. |

Viewport size is different from the document's total scrollable size:

```html
<button type="button" onclick="history.back()">Back</button>
<p id="viewport"></p>
<script>
  function showSize() {
    document.getElementById("viewport").textContent =
      window.innerWidth + " x " + window.innerHeight + " CSS pixels";
  }
  window.addEventListener("resize", showSize);
  showSize();
</script>
```

The example refreshes the dimensions on resize. History navigation requires a corresponding entry; it does not expose an unrestricted list of browsing URLs.

A prompt returns text, so convert numeric input deliberately.

#### Navigator and Browser Detection

`navigator.userAgent`, `navigator.language`, `navigator.onLine` and `navigator.cookieEnabled` report environment information.

`onLine` is only a connectivity indicator, not proof that a particular server is reachable.

`navigator.userAgent.includes("Firefox/")` finds a reported Firefox token. User-agent strings may be reduced, imitated or overlap between browsers.

When checking a capability, feature detection such as `typeof document.querySelector === "function"` directly tests what the program needs.

#### Timing Events

A **timer** schedules a function to run later. Delay is measured in milliseconds; it does not pause the rest of the script.

| Function | Purpose |
| --- | --- |
| `setTimeout(function, delay)` | Schedule one call. |
| `setInterval(function, delay)` | Schedule repeated calls. |
| `clearTimeout(id)` | Cancel the pending one-time timer. |
| `clearInterval(id)` | Cancel the repeating timer. |

The scheduling functions return IDs used for cancellation. Pass a function, not the result of calling it.

```html
<p id="reminder"></p>
<button id="cancelReminder" type="button">Cancel reminder</button>
<p id="countdown"></p>
<script>
  const reminderId = setTimeout(function () {
    document.getElementById("reminder").textContent = "Time to revise DOM";
  }, 2000);
  document.getElementById("cancelReminder").addEventListener("click", function () {
    clearTimeout(reminderId);
    document.getElementById("reminder").textContent = "Reminder cancelled";
  });
  let remaining = 3;
  const intervalId = setInterval(function () {
    document.getElementById("countdown").textContent = remaining;
    remaining--;
    if (remaining === 0) clearInterval(intervalId);
  }, 1000);
</script>
```

The reminder runs once unless cancelled first. The interval displays 3, 2 and 1, then clears itself.

A delay does not guarantee an exact execution time. Cancelling cannot undo a callback that has already run. Deeper scheduling belongs to the planned advanced material.

#### Cookies

A **cookie** is a small name/value item stored by the browser for a site and included in applicable HTTP requests.

It can retain a preference or a session identifier across independent HTTP exchanges. It does not inherently contain the whole session and is not a JavaScript object.

Assigning to `document.cookie` sets one accessible cookie. Reading it returns accessible `name=value` pairs separated by semicolons.

| Attribute | Meaning |
| --- | --- |
| `max-age` | Lifetime in seconds. |
| `expires` | Expiry date in UTC. |
| `path` | Applicable URL path. |

Without `max-age` or `expires`, the cookie is normally a session cookie. Updating or deleting requires the same name/path/domain combination; deletion uses an expired lifetime.[^cookies][^cookie-mechanism]

```javascript
// Create, read and update a preference.
document.cookie = "student=" + encodeURIComponent("Anu Rao") + "; max-age=3600; path=/";
console.log(document.cookie);
document.cookie = "student=" + encodeURIComponent("Mina") + "; max-age=3600; path=/";

function readCookie(name) {
  const prefix = encodeURIComponent(name) + "=";
  for (const part of document.cookie.split(";")) {
    const pair = part.trim();
    if (pair.startsWith(prefix)) return decodeURIComponent(pair.slice(prefix.length));
  }
  return null;
}
console.log(readCookie("student")); // Mina, if this context permits the cookie.
// Delete the same name/path cookie.
document.cookie = "student=; max-age=0; path=/";
```

Encoding keeps spaces and separators from corrupting cookie syntax. The reading function finds the name and decodes its value; the last assignment deletes the matching cookie.

Server-set `HttpOnly` cookies are not exposed through `document.cookie`. Browser settings and context can restrict cookies, so use a served page for practical examples rather than assuming identical behaviour for `file:` pages.

`navigator.cookieEnabled` reports an indicator, not a guarantee that every context accepts every cookie.[^cookie-indicator]

> A persistent cookie has a specified lifetime. A persistent HTTP connection reuses a connection. These are separate mechanisms.

#### Examination Practice

**1. Create and delete cookies in JavaScript, with examples.** (3 marks)[^pyq-41]

**Answer.** A cookie stores a small name/value pair for a site. Assign `document.cookie = "user=Anu; max-age=3600; path=/";` to create a one-hour cookie. Read accessible cookies using `document.cookie`. Delete the same cookie using `document.cookie = "user=; max-age=0; path=/";`, or by setting an expiry date in the past with the same path/domain.

**2. Define cookies and write a function displaying whether they are enabled.** (2 marks)[^pyq-42]

**Answer.** Cookies are small name/value items stored by the browser for a site and included in applicable HTTP requests, allowing state to be retained.

```html
<p id="cookieStatus"></p>
<script>
  function showCookieStatus() {
    document.getElementById("cookieStatus").textContent =
      navigator.cookieEnabled ? "Cookies enabled" : "Cookies disabled";
  }
  showCookieStatus();
</script>
```

This reports the navigator indicator requested by the exam; it is not a test that every storage context accepts cookies.

#### Additional Practice

**Original question · Suggested 5 marks.** Distinguish DOM from BOM, distinguish NodeList membership from node contents, and show a cancellable one-time reminder.

**Answer.** DOM represents/manipulates document nodes; BOM refers to browser-environment facilities such as window/history/timers. `querySelectorAll()` returns a static NodeList: later additions do not change its membership, but its node objects still reflect edits. A reminder can be written as `const id = setTimeout(function () { alert("Revise"); }, 2000);` and cancelled before execution using `clearTimeout(id)`. Here, 2000 is a delay in milliseconds; cancellation does not undo a callback that has already executed.

---

> **Planned material.** Advanced JavaScript and Module III retain their syllabus outlines. Teaching and model answers for these topics are still to be written.

### II.15 Closures, hoisting and IIFEs

**Planned coverage.** Closures; hoisting; immediately invoked function expressions (IIFEs); necessary scope context.

**Question index.** No direct question found. `let`/`var` in [2024 paper, image 2][paper-2024-b], Q4(c), is a related foundation, not an explicit closures/hoisting/IIFE question.

*Reference support:* supplied references do not develop these topics.

### II.16 Event loops, callbacks, promises and async/await

**Planned coverage.** Event loops; callback functions; promises; async/await.

**Question index.**

- **I | [2023 collection][paper-2023], Q2(b), p. 1, 5 marks overall:** Synchronous/asynchronous requests, callback role in AJAX and XHR ready states. Retain its callback/asynchronous-programming application here; full AJAX mapping is III.7.

*Reference support:* `Sources/nodejs.pdf`, pp. 10-24, supports callbacks/event loops. No direct promises/async-await question found and reference coverage is limited.

### II.17 Functional programming and reusable computation

**Planned coverage.** Functional programming; higher-order functions; generators; memoization; currying.

**Question index.** No direct question found.

*Reference support:* supplied references do not adequately develop these topics.

## Module III - XML, AJAX, jQuery, JSON, React.js and Node.js (12 hours)

The following topics retain their syllabus outline and verified question index. Teaching material and model answers are still to be written.

### III.1 XML introduction, HTML comparison and document structure

**Planned coverage.** XML introduction; XML vs HTML; structure of an XML document; well-formed XML documents.

**Question index.**

- **I | [2021 paper, 2018-22 batch][paper-2021-a], Q5(ii-iii), p. 3, marks not visible:** Provide a sample pizza-order XML message for the DTD and identify its building blocks. Q5(i) is indexed in III.2.
- **I | [2021 paper, 2019-23 batch][paper-2021-b], Q3(i), p. 2, marks not visible:** Well-formed vs valid XML, with DTD/types/examples; shared with III.2.
- **I | [2021 paper, 2019-23 batch][paper-2021-b], Q3(ii), p. 2, marks not visible:** Book-catalogue DTD, corresponding XML message and explanation of XML structure; shared with III.2.

*Reference support:* `Sources/XML.pdf`, pp. 1-7.

### III.2 DTD, XML validation and valid documents

**Planned coverage.** Document Type Declaration (DTD), as worded in the syllabus, and DTD definitions; internal/external DTD; XML validation; valid XML document; relation to well-formedness.

**Question index.**

- **I | [2019 paper][paper-2019], Q2.B(i), p. 1, 1+2 marks:** Define DTD and write one for the supplied `book` XML with `id`, title and author.
- **I | [2021 paper, 2018-22 batch][paper-2021-a], Q5(i-iii), p. 3, marks not visible:** Pizza-order message DTD, a sample XML message and its building blocks.
- **I | [2021 paper, 2019-23 batch][paper-2021-b], Q3(i), p. 2, marks not visible:** Well-formed/valid XML; DTD and its types with examples.
- **I | [2021 paper, 2019-23 batch][paper-2021-b], Q3(ii), p. 2, marks not visible:** DTD and sample book-catalogue XML message with a structural explanation.

*Reference support:* `Sources/XML.pdf`, pp. 8-11.

### III.3 XML namespaces and XML Schema

**Planned coverage.** XML Namespaces; XML Schema; schema-based validation and its relation to DTD.

**Question index.** No direct question found. Generic XML validity questions above do not establish a Schema-specific occurrence.

*Reference support:* `Sources/XML.pdf`, pp. 7-11.

### III.4 XML DOM

**Planned coverage.** XML DOM and working with XML document nodes.

**Question index.** No direct XML-DOM question found. HTML DOM questions are indexed separately in Module II.

*Reference support:* only a brief mention in `Sources/XML.pdf`, p. 6; AJAX XML example in `Sources/AJAX.pdf`, pp. 4-7, offers limited context.

### III.5 XSL and XSL transformation

**Planned coverage.** XSL; XSL Transformation; necessary XML selection/template context within transformation examples.

**Question index.** No direct question found.

*Reference support:* `Sources/XML.pdf`, pp. 12-15, is introductory and lacks a complete worked transformation.

### III.6 AJAX introduction and client-server flow

**Planned coverage.** AJAX introduction; relation to JavaScript; asynchronous page updates; client-server architecture as context for AJAX operation.

**Question index.**

- **I | [2019 paper][paper-2019], Q4.A, p. 2, 1+2+2 marks:** Define AJAX, explain implementation steps and important XMLHttpRequest properties/methods. Same question as III.7.
- **P | [2023 collection][paper-2023], Q4(b), p. 1, 2.5 marks per selected short note:** Retain JSON and AJAX options only; exclude Spring and Struts. Answer-any-two choice applies to the original question.
- **I | [2023 collection][paper-2023], Group A Q1(a), p. 3, 2 marks:** Difference between AJAX and JavaScript.
- **P | [2023 collection][paper-2023], Group B Q2, p. 3, 10 marks overall:** Retain JSON in AJAX and the AJAX client-server architecture diagram/explanation. The “types of post backs” clause is U; no retained-only marks are specified.

*Reference support:* `Sources/AJAX.pdf`, pp. 1-4.

### III.7 XMLHttpRequest and server responses

**Planned coverage.** XMLHttpRequest object; request creation/sending; properties/methods; server response; ready states; response handlers and synchronous/asynchronous request context.

**Question index.**

- **I | [2019 paper][paper-2019], Q4.A, p. 2, 1+2+2 marks overall:** AJAX definition/steps and important XHR properties/methods; the final portion is 2 marks.
- **I | [2023 collection][paper-2023], Q1(b), p. 1, 2 marks:** Different ready states of an AJAX request.
- **I | [2023 collection][paper-2023], Q2(b), p. 1, 5 marks:** Synchronous/asynchronous requests, role of AJAX callbacks and request ready states.

*Reference support:* `Sources/AJAX.pdf`, pp. 3-6, illustrates a completion/status check; expand to the full prescribed object/response topic later.

### III.8 AJAX with XML and databases

**Planned coverage.** AJAX XML; AJAX Database; browser request, endpoint response and DOM-update flow in these examples.

**Question index.** No direct question specifically requiring AJAX XML/database integration found. Do not relabel old JDBC or servlet database questions as AJAX questions.

*Reference support:* `Sources/AJAX.pdf`, pp. 4-7 (XML), pp. 7-11 (database). Keep PHP as incidental example code, not a separate study unit.

### III.9 jQuery DOM selectors

**Planned coverage.** jQuery DOM selectors; selecting elements by the prescribed DOM-selection context.

**Question index.**

- **I | [2023 collection][paper-2023], Group A Q1(b), p. 3, 2 marks:** Explain jQuery DOM selectors.

*Reference support:* `Sources/jQuery.pdf`, pp. 4-8, 12. Check legacy selector examples when writing.

### III.10 jQuery HTML content and HTML DOM

**Planned coverage.** HTML Content; jQuery HTML DOM; reading/changing content and attributes; adding/removing DOM elements.

**Question index.**

- **P | [2023 collection][paper-2023], Group B Q4, p. 3, 10 marks overall:** Retain built-in functions that operate on the prescribed selectors/HTML content/CSS/DOM, plus the background-color task in III.11. Do not expand this broad wording into every jQuery API; no subpart split is printed.
- **U - REVIEW | [2023 collection][paper-2023], Q1(e), p. 1, 2 marks:** Explain `ready()`, `mouseenter()`, `mousedown()` and `toggle()`. These span DOM readiness, event helpers and effects; jQuery events/effects are not explicitly among the four prescribed jQuery topics. Confirm intended depth before including the item as required study.

*Reference support:* `Sources/jQuery.pdf`, pp. 8, 10-11, 14. Its event/effect pages do not resolve the syllabus boundary.

### III.11 jQuery CSS styles

**Planned coverage.** jQuery CSS Styles; reading/updating styles and classes in the DOM context.

**Question index.**

- **P | [2023 collection][paper-2023], Group B Q4, p. 3, 10 marks overall:** Write jQuery code to get the background color of an element in a paragraph; explain only syllabus-aligned built-in functions. Same question as III.10; background-color-only marks are not printed.

*Reference support:* `Sources/jQuery.pdf`, pp. 9, 14; check the reference's method-name typos during later writing.

### III.12 JSON introduction, syntax, types, objects and arrays

**Planned coverage.** Introduction; syntax; data types; Object; Arrays; nesting and structured data as applications of these topics.

**Question index.**

- **I | [2021 paper, 2018-22 batch][paper-2021-a], Q5(iv-v), p. 3, marks not visible:** What JSON is and four reasons for preferring JSON over XML for message exchange. Treat the requested preference as a question to assess, not a universal rule.
- **P | [2023 collection][paper-2023], Q4(b), p. 1, 2.5 marks per selected short note:** JSON option only here; AJAX is indexed in III.6; Spring/Struts are excluded.
- **I | [2023 collection][paper-2023], Group A Q1(c), p. 3, 2 marks:** JSON syntax and its data types.
- **P | [2023 collection][paper-2023], Group B Q2, p. 3, 10 marks overall:** JSON in AJAX, within the retained portion; exclude unresolved postback taxonomy.
- **I | [2023 collection][paper-2023], Group B Q3, p. 3, 10 marks:** JSON applications and a program storing five books by author, topic and edition.

*Reference support:* `Sources/JSON.pdf`, pp. 2-31, filtered to the current syllabus.

### III.13 JSON parsing and stringify

**Planned coverage.** Parsing JSON; stringifying JavaScript values; connections to JSON objects/arrays and data interchange.

**Question index.** No direct parsing/stringifying question found. The five-book question does not explicitly demand either operation.

*Reference support:* supplied JSON reference does not develop these operations.

### III.14 React introduction, ES6, rendering HTML and JSX

**Planned coverage.** React.js introduction; ES6; Render HTML; JSX.

**Question index.**

- **I | [2023 collection][paper-2023], Q2(a), p. 1, 5 marks overall:** Creator/company, React advantages, props, JSX and how JSX works with an example. Props are also indexed in III.15; no subpart marks are printed.
- **I | [2023 collection][paper-2023], Group A Q1(d), p. 3, 2 marks:** What ES6 and JSX are.

*Reference support:* `Sources/reactjs.pdf`, pp. 2, 4-17, 22. Develop ES6 coverage within the syllabus rather than importing the reference's separate course outline.

### III.15 React components, props, state and lifecycle

**Planned coverage.** Components; props; state; lifecycle. The stray “2” before “Lifecycle” in the scan does not define an additional topic.

**Question index.**

- **I | [2023 collection][paper-2023], Q2(a), p. 1, 5 marks overall:** Props portion of the React/JSX question; same question as III.14.

*Reference support:* `Sources/reactjs.pdf`, pp. 18-21, supports components/props. State and lifecycle need further support. No direct state/lifecycle question found.

### III.16 React events, forms and CSS

**Planned coverage.** Events; forms; CSS in React.

**Question index.** No direct React-specific question found. Plain HTML/JavaScript form questions are not React occurrences.

*Reference support:* supplied React reference does not develop these prescribed topics.

### III.17 Node.js introduction and development environment

**Planned coverage.** Introduction to Node.js; setup development environment; running an application as setup context.

**Question index.**

- **I | [2023 collection][paper-2023], Q1(a), p. 1, 2 marks overall:** What Node.js can do and syntax to include the HTTP module; module portion also indexed in III.18.

*Reference support:* `Sources/nodejs.pdf`, pp. 1-5; installation/setup coverage needs supplementation.

### III.18 Node.js modules and npm

**Planned coverage.** Node.js modules; Node Package Manager; importing/exporting modules and managing packages as applications of these topics.

**Question index.**

- **I | [2023 collection][paper-2023], Q1(a), p. 1, 2 marks overall:** Include the HTTP module in a Node.js program; same item as III.17.
- **I | [2023 collection][paper-2023], Group A Q1(e), p. 3, 2 marks:** Explain Node.js modules.

*Reference support:* `Sources/nodejs.pdf`, pp. 6-9, 17. No direct npm question found.

### III.19 Node.js web server and events

**Planned coverage.** Creating a web server; Events; event-driven request handling.

**Question index.**

- **I | [2023 collection][paper-2023], Q1(a), p. 1, 2 marks overall:** Node capabilities and including HTTP are relevant foundations; the actual prompt does not ask for a complete server implementation.

*Reference support:* `Sources/nodejs.pdf`, pp. 13-24, 29-33. No direct full-server or EventEmitter implementation question found.

### III.20 Node.js file system and debugging

**Planned coverage.** File System; Debugging Node.js Application.

**Question index.** No direct Node-specific file-system/debugging question found. The 2024 browser JavaScript debugging item belongs to [Loops, Regular Expressions and Errors](#9-loops-regular-expressions-and-errors).

*Reference support:* `Sources/nodejs.pdf`, pp. 34-42, covers file operations. Debugging needs further support.

### III.21 Express.js and serving static resources

**Planned coverage.** Express.js; Serving Static Resources.

**Question index.** No direct question found.

*Reference support:* the npm reference installs Express as an example but does not teach Express or static serving.

### III.22 Node.js database connectivity

**Planned coverage.** Database Connectivity in the Node.js context.

**Question index.** No direct Node.js database-connectivity question found. Older Java/JDBC/servlet questions remain excluded; do not rewrite them into invented Node questions.

*Reference support:* supplied Node reference does not cover this topic.

## Appendix A - Examination evidence

**Scope authority:** [IWT-7th-Semester-IT.pdf](../Syllabus/IWT-7th-Semester-IT.pdf), Module I on PDF p. 2, Module II on pp. 2-3, Module III on p. 3. Completed teaching covers Module I and JavaScript fundamentals, DOM and BOM in Sections 1-12; advanced JavaScript and Module III retain their planned outlines.

| Study priority | Evidence in the supplied questions | Planning decision |
| --- | --- | --- |
| First: HTML/CSS construction and JavaScript forms/DOM | HTML/CSS tasks occur in the 2019 paper, both 2021 files and the 2024 images: **4 source sets**. Explicit form-validation tasks occur in both 2021 files and 2024: **3 sets**. The 2018-22 batch travel task carries 10 points; 2024 website tasks carry 5 marks each. | Practise page construction, styling, validation and DOM updates together after learning their foundations. |
| First: XML structure, well-formedness, validity and DTD design | DTD questions occur in 2019 and both 2021 files: **3 sets**. The 2019 DTD task is 1+2 marks; the 2021 questions require complete XML/DTD examples, but their marks are not visible. | Prepare both short distinctions and document-design answers. |
| Next: AJAX and JSON | AJAX occurs in 2019 and the dated 2023 page; JSON occurs in the 2018-22 batch file and the dated 2023 page. Both also occur on the separate undated sheet in the 2023 PDF. AJAX implementation/XHR is 5 marks in 2019; the dated 2023 synchronous/asynchronous AJAX question is 5 marks. | Prioritise request flow, XHR states/responses, JSON syntax and structured data; retain all prescribed parsing/stringifying topics. |
| Next: jQuery, React and Node.js | Each appears on the dated 2023 page and the separate undated sheet. The latter offers 10-mark JSON and jQuery questions; dated 2023 React is 5 marks and Node.js is 2 marks. Some jQuery wording needs scope review. | Prepare the exact Module III topics. These two pages are different assessment fragments, not two verified exam years. |
| Required despite little/no direct past-question evidence | HTTP details, web history/standards/cyber laws, many CSS properties, advanced JavaScript, XML Schema/XSL/XML DOM, React state/lifecycle/forms/CSS, Node debugging/Express/static resources/database connectivity. | Keep every prescribed subtopic. Lack of observed questions is not evidence of exclusion. |

**Frequency means observed presence, not predicted probability.** Counts above are by source set, once per topic family per set, across 2019, the two 2021 files, the dated 2023 page and the paired 2024 images. The undated sheet is reported separately. Repeated references to one question below do not add occurrences. Partial questions count only for their retained topic; uncertain items do not support frequency claims. These are small, uneven and sometimes incomplete samples from different semesters/branches.

**Observed formats:** 2019 is 50 marks/2.5 hours, any five of six 10-mark questions, with 1-3 mark subparts and 2.5-mark short notes. The dated 2023 page says 50 marks/2.5 hours and answer all, but only 40 marks are visible. The undated sheet has five compulsory 2-mark items and any two of three 10-mark questions. The paired November 2024 images show 50 marks/2.5 hours, answer all, arranged as five 10-mark blocks: Q1/Q3 have two 5-mark tasks; Q2/Q4/Q5 have five 2-mark items. The 2021 screenshot collections do not establish a complete timed-paper format.

**Prediction boundary:** the study order above is a judgement from coverage, recurrence and task size, not a forecast of the next paper or a promised marks distribution. The current syllabus allocates **8/10/12 teaching hours** to Modules I/II/III, respectively (30 total); this is not an exam-weightage rule. The supplied 2024 paper tests Modules I-II, which does not remove Module III from the current syllabus. Syllabus p. 4 assigns the end-term exam 50% of course evaluation, not a guaranteed question pattern.

## Appendix B - Sources and reference limits

- **I = in scope:** the asked task fits a prescribed topic.
- **P = partially in scope:** study only the explicitly retained portion; the rest is excluded in the audit.
- **O = out of scope:** recorded only in the audit, with no planned study section.
- **U = uncertain; REVIEW:** wording/context does not establish syllabus alignment. Do not treat it as prescribed or count it as recurrence.
- Question descriptions are compact paraphrases of actual prompts, not invented practice questions. `Not visible` means the source does not verify the marks; no equal split is assumed for a multipart question.
- PDF page numbers are **1-based file pages**, not inferred printed page labels. For PNGs, use the image filename; their page order is inferred from question continuity and is not printed pagination.
- A section's **Planned coverage** paragraph identifies material still to be written. **No direct question found** records an evidence gap, not permission to skip the section. Cross-topic questions may appear in more than one section.

| Supplied paper file | Identity and limitations |
| --- | --- |
| [CS201_Internet_and_Web_Technology_April_2019.pdf](../Papers/CS201_Internet_and_Web_Technology_April_2019.pdf) | 2 PDF pages; April 2019, sixth semester, EEE, CS201. Older course context; filter against the present IT syllabus. |
| [IWT 2021 Endsem (2018-22 batch).pdf](<../Papers/IWT 2021 Endsem (2018-22 batch).pdf>) | 3 PDF pages of screenshots. Year/batch are supplied by filename. Travel prompt has no printed question number; vaccination form is identified as q2 in its screenshot title. Q3-Q5 are numbered. The travel prompt shows 10 points; the vaccination mark label is clipped. |
| [IWT 2021 Endsem (2019-23 batch).pdf](<../Papers/IWT 2021 Endsem (2019-23 batch).pdf>) | 5 PDF pages; Q1-Q5 on pp. 1-4, p. 5 blank. Year/batch are supplied by filename. Marks are not visible. Q5(ii)'s final instruction is cut off; its HTML/servlet requirement is readable. |
| [IWT_Question_Papers_2023.pdf](../Papers/IWT_Question_Papers_2023.pdf), p. 1 | Dated November 2023, seventh semester IT, IWT II. Header declares two pages and 50 marks, but the file does not contain a coherent continuation: only Q1-Q4, totalling 40 marks, are visible here. Missing questions cannot be reconstructed. |
| Same PDF, p. 2 | Android fragment: XML checkboxes, contextual menu, implicit intents and ConstraintLayout. It is not a continuation consistent with the IWT questions on p. 1. Excluded from IWT frequency. |
| Same PDF, p. 3 | Separate undated sheet headed Internet and Web Technologies-2, seventh semester IT, Groups A/B, 30 attempted marks. Do not assign it a verified 2023 exam date just because of the container filename. |
| [A_1.png](../Papers/A_1.png) and [A_2.png](../Papers/A_2.png) | Paired November 2024 end-semester paper, IT. A_1 has Q1 and Q2(a-b); A_2 continues Q2(c-e), then Q3-Q5. One paper for recurrence counting. |

### Reference material: use and gaps

All six supplied references were read. They support later writing but cannot add syllabus topics. Page ranges below refer to PDF file pages.

| Reference | Useful syllabus-aligned coverage | Gaps/boundaries for later writing |
| --- | --- | --- |
| [XML.pdf](../Sources/XML.pdf), 15 pp. | Introduction/HTML comparison pp. 1-5; structure pp. 5-7; namespaces pp. 7-8; DTD pp. 8-10; Schema/validation pp. 10-11; XSL/transformation overview pp. 12-15. | XML DOM is only briefly mentioned (p. 6); worked DOM/XSLT coverage is insufficient. Android/configuration examples do not prescribe Android development. |
| [AJAX.pdf](../Sources/AJAX.pdf), 11 pp. | Introduction/architecture pp. 1-2; XHR/callback/response example pp. 3-4; XML interaction pp. 4-7; database interaction pp. 7-11. | PHP/MySQL are example endpoints, not independent prescribed PHP/JDBC topics. XHR states need fuller treatment than the sample's completion check. |
| [jQuery.pdf](../Sources/jQuery.pdf), 16 pp. | Basics/selectors pp. 4-7, 12; chaining p. 8; styles p. 9; markup pp. 10, 14; DOM overview p. 11. | Events/effects on pp. 13, 15 do not expand the four listed jQuery syllabus topics. Some spellings/examples need checking when writing, including `ccs` on p. 9 and `xss` on p. 14. |
| [JSON.pdf](../Sources/JSON.pdf), 33 pp. | Introduction pp. 2-5; syntax/objects/arrays pp. 7-11; types pp. 13-19; nesting pp. 20-22; XML comparison pp. 24-31. | Does not teach the prescribed JavaScript parsing/stringifying operations. Java/Gson material on p. 32 is not prescribed. |
| [reactjs.pdf](../Sources/reactjs.pdf), 26 pp. | Intro/rendering pp. 2, 4-12; JSX pp. 13-17; components/props pp. 18-21; Babel p. 22. | State, lifecycle, events, forms and React CSS are not developed. The reference's own “Syllabus” on p. 3 is not the official course scope; do not import Router, virtualized lists, webpack or fetch-hook topics from it. |
| [nodejs.pdf](../Sources/nodejs.pdf), 42 pp. | Intro/running/REPL pp. 1-5; npm/modules pp. 6-9; callbacks pp. 10-12; event loop/events pp. 13-24; HTTP server pp. 29-33; file system pp. 34-42. | Installation detail is limited; debugging, Express, static serving and database connectivity are missing. Streams pp. 25-29 may support file/server examples but are not a separate syllabus unit. |

The supplied references do not cover most Module I material or most advanced JavaScript. The completed foundations use the supplementary sources listed in Appendix D where needed. Any support added for planned sections must remain within the official syllabus.

## Appendix C - Past-paper scope audit

The register below accounts for every visible question, including excluded material. **I** items are described in the notes or planned outline; the boundary tables explain every **P/O/U** decision. Group labels and page numbers distinguish identically numbered questions in the mixed 2023 file.

### Complete classification register

| File / page | I: in scope | P: retain only aligned portions | O: exclude | U: review |
| --- | --- | --- | --- | --- |
| `CS201_Internet_and_Web_Technology_April_2019.pdf`, pp. 1-2 | Q1.A(i-ii), Q1.B(i-ii); Q2.A(i-ii), Q2.B(i-ii); Q4.A; Q5.B(ii); Q6.A(ii) | Q3.A(i), Q3.B, Q6.A(i) | Q3.A(ii), Q4.B, Q5.A(i), Q5.B(i), Q6.B(i), Q6.B(iii) | Q5.A(ii) |
| `IWT 2021 Endsem (2018-22 batch).pdf`, pp. 1-3 | Unnumbered travel prompt; Q2; Q5(i-v) | None | Q3, Q4 | None; travel numbering and Q2 marks are provenance limitations |
| `IWT 2021 Endsem (2019-23 batch).pdf`, pp. 1-4 | Q1(i); Q2(i-ii); Q3(i-ii) | Q1(ii), Q5(ii) | Q4(i-ii), Q5(i) | None; Q5(ii)'s closing instruction is cut off |
| `IWT_Question_Papers_2023.pdf`, p. 1, dated exam | Q1(a-b), Q2(a-b) | Q4(b) | Q1(c), Q3(a-b), Q4(a) | Q1(d-e) |
| Same PDF, p. 2, Android fragment | None | None | Checkbox prompt; contextual-menu prompt; Q2(a-b) | Checkbox subpart label is obscured; contextual-menu label appears to be (j). Identity/continuity needs review, but the Android tasks are excluded. |
| Same PDF, p. 3, undated Group A/B sheet | Group A Q1(a-e); Group B Q3 | Group B Q2, Q4 | None | Group B Q2's “types of post backs” clause only; parent question classified P |
| `A_1.png` and `A_2.png`, paired 2024 paper | Q1(a-b); Q2(a-e); Q3(a-b); Q4(a-e); Q5(a-e) | None | None | None |

### Partially in-scope questions: exact retention boundaries

| File, locator, verifiable marks | Retain for this outline | Exclude or hold for review |
| --- | --- | --- |
| `CS201_Internet_and_Web_Technology_April_2019.pdf`, Q3.A(i), p. 1, 2+1 | GET vs POST (1 mark), [HTTP: URLs, Messages and Connections](#2-http-urls-messages-and-connections). | JSP tags and examples (2 marks). JSP is not prescribed. |
| Same file, Q3.B, p. 2, 1+4 overall | Explicit HTML registration-form portion, [HTML: Structure, Content and Forms](#3-html-structure-content-and-forms). No HTML-only mark split given. | Servlet lifecycle, servlet code, deployment descriptor and database-table implementation. Node database connectivity does not prescribe this Java stack. |
| Same file, Q6.A(i), p. 2, 1+1 | JavaScript introductory characteristics (from the comparison) and `script` attributes, [HTML: Structure, Content and Forms](#3-html-structure-content-and-forms)/[JavaScript Basics](#6-javascript-basics). Script-attributes portion is 1 mark; do not claim a retained-only total for the comparison. | Detailed Java-language study from the Java-vs-JavaScript comparison. |
| `IWT 2021 Endsem (2019-23 batch).pdf`, Q1(ii), p. 1, marks not visible | CSS purpose, box model, colors, alignment/layout, iframe and inline styles, [HTML: Structure, Content and Forms](#3-html-structure-content-and-forms)/[CSS: Styling and Page Layout](#4-css-styling-and-page-layout)/[CSS: Styling and Page Layout](#4-css-styling-and-page-layout)/[CSS: Styling and Page Layout](#4-css-styling-and-page-layout)/[CSS: Styling and Page Layout](#4-css-styling-and-page-layout). | The diagram's `marquee` instruction. Frames and CSS animation do not explicitly prescribe this old HTML tag. |
| Same file, Q5(ii), p. 4, marks not visible | Explicit HTML login page (`index.html`), [HTML: Structure, Content and Forms](#3-html-structure-content-and-forms). | Servlet authentication/code and `web.xml` mapping. Do not infer the cut-off final instruction. |
| `IWT_Question_Papers_2023.pdf`, Q4(b), p. 1, 2.5+2.5 for any two options | JSON and AJAX short notes, III.6/III.12. | Spring framework and Struts options. Original choice rule does not make them prescribed. |
| Same file, Group B Q2, p. 3, 10 overall | JSON in AJAX and client-server architecture diagram/explanation, III.6/III.12. No marks assigned to retained clauses separately. | “Types of post backs in AJAX” is U: intended taxonomy/technology is unspecified. Do not add ASP.NET postback material without clarification. |
| Same file, Group B Q4, p. 3, 10 overall | Built-in jQuery functions for selectors, HTML content, CSS styles and HTML DOM; requested background-color getter, III.10/III.11. | Unbounded coverage of all jQuery built-ins, including effects/plugins and library-specific event APIs not explicitly prescribed. No separate getter marks are printed. |

### Out-of-scope questions and options

| File and locator | Actual asked material | Why excluded |
| --- | --- | --- |
| `CS201_Internet_and_Web_Technology_April_2019.pdf`, Q3.A(ii), p. 2, 1+1 marks | JSP vs servlet; use of a JDBC driver. | Neither Java server technology nor JDBC is prescribed. |
| Same file, Q4.B, p. 2, 2+3 marks | Five steps connecting Java to a database and a JDBC program. | Node database connectivity is a different prescribed context. |
| Same file, Q5.A(i), p. 2, 1+2 marks | Class vs interface and a class with multiple constructors. | Java context, not a verified JavaScript-class task. |
| Same file, Q5.B(i), p. 2, 1+1+1 marks | JavaBean example, serializable interface and EJB types. | JavaBeans/EJB/Java serialization have no module-level topic. The general outcome's Java Beans mention does not prescribe this detailed Java stack; flag the wording discrepancy for review below. |
| Same file, Q6.B(i), p. 2, 2.5 marks | EJB vs RMI. | Neither is a prescribed module topic. |
| Same file, Q6.B(iii), p. 2, 2.5 marks | VBScript short note. | VBScript is not prescribed. The second listed short-note label is printed “iii”; do not renumber it to “ii”. |
| `IWT 2021 Endsem (2018-22 batch).pdf`, Q3, p. 2, marks not visible | Java Person -> StudentIIIT -> MTechStudent/BTechStudent/PhDStudent classes; accessors/mutators, constructors, `this`/`super`. | Explicit Java program, not JavaScript inheritance. |
| Same file, Q4, p. 2, marks not visible | Health-camp servlet that processes height/weight/health data, displays BMI/status and stores data in a database. | Explicit servlet implementation; there is no separately requested HTML-file subtask to retain. Do not transform it into a new JavaScript/Node question. |
| `IWT 2021 Endsem (2019-23 batch).pdf`, Q4(i), p. 3, marks not visible | Java command-line arithmetic program handling arithmetic exceptions. | Explicit Java, not the prescribed JavaScript errors topic. |
| Same file, Q4(ii), p. 3, marks not visible | Implement Java Item/MP3/DVD/Book inheritance diagram and create objects/attributes. | Explicit Java, not JavaScript classes. |
| Same file, Q5(i), p. 4, marks not visible | Java/JDBC connection; create a students table, insert two rows, display contents. | Java/JDBC stack is not prescribed; do not count as Node connectivity. |
| `IWT_Question_Papers_2023.pdf`, Q1(c), p. 1, 2 marks | J2EE concepts/types. | J2EE is not prescribed. |
| Same file, Q3(a), p. 1, 2+1+2 marks | EJB definition, types and differences. | EJB is not prescribed. |
| Same file, Q3(b), p. 1, 2+2+1 marks | Hibernate architecture, components and object states. | Hibernate is not prescribed. |
| Same file, Q4(a), p. 1, 1+2+2 marks | Microservices, advantages and design best practices with examples. | Microservices-specific architecture/design is not a named module topic; broad “web architecture” does not justify importing this full unit. |
| Same file, Q4(b)(i-ii), p. 1, 2.5 per selected note | Spring framework and Struts. | These are excluded options in the partially retained short-note question. |
| Same file, p. 2, checkbox prompt, label obscured, marks not visible | XML UI checkboxes for chocolate syrup, sprinkles and crushed nuts. | Android layout task in the Android fragment, not web XML/DTD. |
| Same file, p. 2, contextual-menu prompt, label appears (j), marks not visible | Define a contextual menu and steps to create it. | Android context established by adjacent UI and Activity/manifest tasks; not a CSS navigation-menu question. |
| Same file, Q2(a), p. 2, 5 marks | Android layout/Java/manifest for one Activity opening a website/map and sharing text, using editable default-valued fields. | Android Activity/intents/manifest development is not prescribed. Opening a website does not make it an IWT web-development question. |
| Same file, Q2(b), p. 2, 5 marks | Android Hello Constraint app using ConstraintLayout, with layout/Java/manifest files. | Android ConstraintLayout is not CSS layout. |

### Uncertain classifications and source issues to review

| Item | Uncertainty | Current treatment |
| --- | --- | --- |
| `CS201_Internet_and_Web_Technology_April_2019.pdf`, Q5.A(ii), p. 2, 2 marks | “Exception handling” does not name its language; surrounding questions are Java-oriented. | U; held out of required question frequency. Retain only if confirmed to ask JavaScript error handling. |
| `IWT_Question_Papers_2023.pdf`, Q1(d), p. 1, 2 marks | MVC's relation to generic prescribed web architecture versus an older framework-specific unit. | U; retained in this audit for review, with no MVC/framework expansion. |
| Same file, Q1(e), p. 1, 2 marks | jQuery readiness/event/effect methods exceed the four explicitly listed jQuery topics; intended DOM-topic depth is unclear. | U; listed in III.10 for review, excluded from confirmed frequency. |
| Same file, Group B Q2, p. 3, part of 10 marks | “Types of post backs” may refer to a technology-specific taxonomy absent from the syllabus. | Parent question P; this clause U. Study JSON/AJAX architecture only until clarified. |
| Official syllabus, p. 1, course-outcome reference to Java Beans | Module lists on pp. 2-3 prescribe Node.js server-side programming, with no JavaBeans/JSP/servlet/JDBC unit. | Do not revive old Java-stack questions. Treat the outcome/module discrepancy as review-worthy; the explicit module topic list controls this outline. |
| 2023 PDF composition/date | P. 1 lacks its coherent second page; p. 2 is Android; p. 3 is a different undated IWT sheet. | Keep page-specific attribution. Do not invent the missing 10 marks, merge sheets into one exam, or treat the undated sheet as an independently verified exam year. |
| 2021 screenshot provenance/marks | Dates come from filenames; travel number is absent; vaccination marks are clipped; second file has no visible marks and a truncated Q5(ii) ending. | Use visible locators and “not visible/not verifiable”; do not invent totals, question numbers or mark allocations. |

## Appendix D - Reading Map and Writing Standard

### Completed and Planned Material

Sections 1-12 contain the completed foundations, JavaScript, DOM and BOM notes. The advanced JavaScript and Module III outlines remain planned. The original four practice questions are retained alongside the examination questions.

The following map keeps the earlier syllabus-topic identifiers traceable after related topics have been grouped. It also records the coverage of the completed material without interrupting the teaching.

| Earlier topic | Coverage | Read in |
| --- | --- | --- |
| I.1 | Internet, WWW and their histories | [Web Fundamentals and Architecture](#1-web-fundamentals-and-architecture) |
| I.2 | Websites, web applications, web projects and development issues | [Web Fundamentals and Architecture](#1-web-fundamentals-and-architecture) |
| I.3 | Web architecture, web servers and browsers | [Web Fundamentals and Architecture](#1-web-fundamentals-and-architecture) |
| I.4 | Web protocols, Internet standards, TCP/IP and MIME | [Web Fundamentals and Architecture](#1-web-fundamentals-and-architecture) |
| I.5 | URL anatomy and HTTP messages | [HTTP: URLs, Messages and Connections](#2-http-urls-messages-and-connections) |
| I.6 | Connections, web caching and proxies | [HTTP: URLs, Messages and Connections](#2-http-urls-messages-and-connections) |
| I.7 | Cyber laws | [Web Fundamentals and Architecture](#1-web-fundamentals-and-architecture) |
| I.8 | HTML foundations and document metadata | [HTML: Structure, Content and Forms](#3-html-structure-content-and-forms) |
| I.9 | HTML text, headings and lists | [HTML: Structure, Content and Forms](#3-html-structure-content-and-forms) |
| I.10 | Anchors, images and video | [HTML: Structure, Content and Forms](#3-html-structure-content-and-forms) |
| I.11 | HTML tables | [HTML: Structure, Content and Forms](#3-html-structure-content-and-forms) |
| I.12 | Frames | [HTML: Structure, Content and Forms](#3-html-structure-content-and-forms) |
| I.13 | HTML forms and integrated page construction | [HTML: Structure, Content and Forms](#3-html-structure-content-and-forms) |
| I.14 | Inline, internal and external CSS | [CSS: Styling and Page Layout](#4-css-styling-and-page-layout) |
| I.15 | CSS selectors, combinators, attribute selectors and specificity | [CSS: Styling and Page Layout](#4-css-styling-and-page-layout) |
| I.16 | Pseudo-classes and pseudo-elements | [CSS: Styling and Page Layout](#4-css-styling-and-page-layout) |
| I.17 | Colors and backgrounds | [CSS: Styling and Page Layout](#4-css-styling-and-page-layout) |
| I.18 | Borders, margins, padding and units | [CSS: Styling and Page Layout](#4-css-styling-and-page-layout) |
| I.19 | Fonts, links, icons, lists and table styling | [CSS: Styling and Page Layout](#4-css-styling-and-page-layout) |
| I.20 | Display, positioning and website layout | [CSS: Styling and Page Layout](#4-css-styling-and-page-layout) |
| I.21 | Navigation bars, dropdowns, image galleries and styled forms | [CSS: Styling and Page Layout](#4-css-styling-and-page-layout) |
| I.22 | Opacity, text effects, animations, tooltips and counters | [CSS: Styling and Page Layout](#4-css-styling-and-page-layout) |
| II.1 | JavaScript introduction, statements, comments and bindings | [JavaScript Basics](#6-javascript-basics) |
| II.2 | Functions, objects and `this` | [Functions, Objects and Classes](#7-functions-objects-and-classes) |
| II.3 | Strings, numbers, arrays, Date, Math and random values | [Strings, Numbers, Arrays and Dates](#8-strings-numbers-arrays-and-dates) |
| II.4 | Loops and small programs | [Loops, Regular Expressions and Errors](#9-loops-regular-expressions-and-errors) |
| II.5 | Regular expressions | [Loops, Regular Expressions and Errors](#9-loops-regular-expressions-and-errors) |
| II.6 | Errors and debugging | [Loops, Regular Expressions and Errors](#9-loops-regular-expressions-and-errors) |
| II.7 | JavaScript classes and inheritance | [Functions, Objects and Classes](#7-functions-objects-and-classes) |
| II.8 | HTML DOM foundations, selection and manipulation | [Working with the HTML DOM](#10-working-with-the-html-dom) |
| II.9 | JavaScript and DOM events | [Working with the HTML DOM](#10-working-with-the-html-dom) |
| II.10 | DOM navigation, nodes and NodeList | [Working with the HTML DOM](#10-working-with-the-html-dom) |
| II.11 | Documents, forms and validation | [Forms and Validation](#11-forms-and-validation) |
| II.12 | Browser Object Model | [The Browser Object Model](#12-the-browser-object-model) |
| II.13 | JavaScript timing events | [The Browser Object Model](#12-the-browser-object-model) |
| II.14 | Cookies and working with cookies in JavaScript | [The Browser Object Model](#12-the-browser-object-model) |

### Style for Subsequent Sections

Use [the supplied RecSys notes](https://github.com/souiiii/Everything/blob/main/Midsem/RecSys/unit3_chatgpt.md) as the primary style reference: introduce the core idea simply, develop it in short connected paragraphs, then show how an example works. Use a meaningful subheading when the question being explained changes, and reserve tables, steps and blockquotes for places where they clarify the idea.

Keep examination questions together after a logical topic. Preserve the actual prompt, marks, source and syllabus boundary, and provide a complete answer proportionate to the marks. Put detailed references in source notes. The IWT syllabus, supplied sources and papers remain the content authorities.

### Shared Applications

The following links preserve applications previously repeated under individual HTML/CSS topics; their complete answers are grouped in the page-design section.

- Styled headings and travel-story text are part of the integrated task. The complete model in [Complete Page-Design Questions](#5-complete-page-design-questions) includes both; no separate marks for text formatting are printed.[^pyq-05]
- The diagram includes an iframe with inline styles. The full retained layout answer is in [Complete Page-Design Questions](#5-complete-page-design-questions). Its additional `marquee` requirement remains excluded.[^pyq-11]
- The CSS box-model layout is answered in [Complete Page-Design Questions](#5-complete-page-design-questions). Margins also appear in the 10-point travel task answered in [Complete Page-Design Questions](#5-complete-page-design-questions).[^pyq-19]

### Examination Question Sources

[^pyq-01]: [2019 paper][paper-2019], Q1.A(i), PDF p. 1, 2 marks
[^pyq-02]: [2021 paper, 2019-23 batch][paper-2021-b], Q1(i), PDF p. 1, marks not visible
[^pyq-03]: [2019 paper][paper-2019], Q3.A(i), PDF p. 1, retained GET/POST part: 1 mark; original 2+1 marks · Retained portion
[^pyq-04]: [2019 paper][paper-2019], Q6.A(i), PDF p. 2, script-attributes part: 1 mark; original 1+1 marks · Retained portion
[^pyq-05]: [2021 paper, 2018-22 batch][paper-2021-a], unnumbered travel prompt, PDF p. 1, 10 points overall · Shared application.
[^pyq-06]: [2024 paper, image 1][paper-2024-a], Q2(a), 2 marks
[^pyq-07]: [2024 paper, image 2][paper-2024-b], Q2(e), 2 marks
[^pyq-08]: [2021 paper, 2018-22 batch][paper-2021-a], unnumbered travel prompt, PDF p. 1, 10 points
[^pyq-09]: [2019 paper][paper-2019], Q1.A(ii), PDF p. 1, 3 marks
[^pyq-10]: [2024 paper, image 2][paper-2024-b], Q2(d), 2 marks
[^pyq-11]: [2021 paper, 2019-23 batch][paper-2021-b], Q1(ii), PDF p. 1, marks not visible · Retained portion · Shared application.
[^pyq-12]: [2019 paper][paper-2019], Q3.B, PDF p. 2, original 1+4 marks; no HTML-only split · Retained portion
[^pyq-13]: [2021 paper, 2019-23 batch][paper-2021-b], Q5(ii), PDF p. 4, marks not visible · Retained portion
[^pyq-14]: [2024 paper, image 2][paper-2024-b], Q3(a), 5 marks
[^pyq-15]: [2019 paper][paper-2019], Q1.B(i), PDF p. 1, 2 marks
[^pyq-16]: [2019 paper][paper-2019], Q1.B(ii), PDF p. 1, 3 marks
[^pyq-17]: [2024 paper, image 2][paper-2024-b], Q2(c), 2 marks
[^pyq-18]: [2024 paper, image 1][paper-2024-a], Q2(b), 2 marks
[^pyq-19]: [2021 paper, 2019-23 batch][paper-2021-b], Q1(ii), PDF p. 1, marks not visible · Retained portion · Shared application.
[^pyq-20]: [2024 paper, image 1][paper-2024-a], Q1(a), 5 marks
[^pyq-21]: [2021 paper, 2019-23 batch][paper-2021-b], Q1(ii), PDF p. 1, marks not visible · Retained portion
[^pyq-22]: [2024 paper, image 2][paper-2024-b], Q4(c), 2 marks
[^pyq-23]: [2019 paper][paper-2019], Q6.A(i), PDF p. 2, original 1+1 marks · Retained portion
[^pyq-24]: [2019 paper][paper-2019], Q2.A(i), PDF p. 1, 2 marks
[^pyq-25]: [2024 paper, image 2][paper-2024-b], Q4(a), 2 marks
[^pyq-26]: [2019 paper][paper-2019], Q5.B(ii), PDF p. 2, 1+1 marks
[^pyq-27]: [2024 paper, image 2][paper-2024-b], Q4(d), 2 marks
[^pyq-28]: [2024 paper, image 2][paper-2024-b], Q4(e), 2 marks
[^pyq-29]: [2019 paper][paper-2019], Q2.B(ii), PDF p. 1, 2 marks
[^pyq-30]: [2019 paper][paper-2019], Q6.A(ii), PDF p. 2, 3 marks
[^pyq-31]: [2024 paper, image 2][paper-2024-b], Q4(b), 2 marks
[^pyq-32]: [2024 paper, image 2][paper-2024-b], Q5(d), 2 marks
[^pyq-33]: [2024 paper, image 2][paper-2024-b], Q3(b), 5 marks
[^pyq-34]: [2024 paper, image 2][paper-2024-b], Q5(c), 2 marks
[^pyq-35]: [2021 paper, 2019-23 batch][paper-2021-b], Q2(ii), PDF p. 2, marks not visible
[^pyq-36]: [2024 paper, image 2][paper-2024-b], Q5(b), 2 marks for the entire item
[^pyq-37]: [2021 paper, 2019-23 batch][paper-2021-b], Q2(i), PDF p. 2, marks not visible
[^pyq-38]: [2021 paper, 2018-22 batch][paper-2021-a], Q2 (screenshot title), PDF p. 1, marks clipped/not verifiable
[^pyq-39]: [2024 paper, image 1][paper-2024-a], Q1(b), 5 marks
[^pyq-40]: [2024 paper, image 2][paper-2024-b], Q5(e), 2 marks
[^pyq-41]: [2019 paper][paper-2019], Q2.A(ii), PDF p. 1, 3 marks
[^pyq-42]: [2024 paper, image 2][paper-2024-b], Q5(a), 2 marks

### Supplementary source notes

The following links support details already used in the completed notes; they do not enlarge the syllabus.

[^web-history]: CERN, [A short history of the Web](https://home.cern/science/computing/the-birth-of-the-web/short-history-web/).
[^internet-history]: Internet Society, [A Brief History of the Internet](https://www.internetsociety.org/internet/history-internet/brief-history-internet/).
[^http-overview]: MDN, [Overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview).
[^http-messages]: MDN, [HTTP messages](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Messages).
[^cyber-law]: India Code, [Information Technology Act, 2000](https://www.indiacode.nic.in/bitstream/123456789/13116/1/it_act_2000_updated.pdf).
[^frames]: MDN, [The frameset element](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/frameset).
[^specificity]: MDN, [Specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Specificity).
[^bindings]: MDN, [let declarations](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let).
[^validation]: MDN, [Constraint validation](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Constraint_validation).
[^collections]: MDN, [querySelectorAll()](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll).
[^cookies]: MDN, [Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie).
[^cookie-mechanism]: RFC Editor, [HTTP State Management Mechanism, RFC 6265](https://www.rfc-editor.org/info/rfc6265/).
[^cookie-indicator]: MDN, [Navigator.cookieEnabled](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/cookieEnabled).

[paper-2019]: <../Papers/CS201_Internet_and_Web_Technology_April_2019.pdf>
[paper-2021-a]: <../Papers/IWT 2021 Endsem (2018-22 batch).pdf>
[paper-2021-b]: <../Papers/IWT 2021 Endsem (2019-23 batch).pdf>
[paper-2023]: <../Papers/IWT_Question_Papers_2023.pdf>
[paper-2024-a]: <../Papers/A_1.png>
[paper-2024-b]: <../Papers/A_2.png>
