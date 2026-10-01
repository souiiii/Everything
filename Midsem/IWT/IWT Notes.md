# IWT Notes

<aside>
🎯

**Midsem scope:** The Web, HTML and CSS only. JavaScript and later modules are intentionally excluded.

</aside>

---

# 1. The Web: Internet and WWW

The **Internet** is the global network infrastructure that connects computers and smaller networks. It is best understood as a *network of networks*. Devices communicate because they follow agreed rules, especially the TCP/IP protocol suite. The Internet is broader than the Web: email, file transfer, remote access and many other services can use the Internet without being part of the World Wide Web.

The **World Wide Web (WWW)** is one service that runs on top of the Internet. It consists of interlinked resources identified by URLs and accessed mainly through browsers using HTTP or HTTPS. For an exam answer, remember the distinction clearly: **the Internet provides connectivity; the Web provides hyperlinked information and applications over that connectivity**.

## 1.1 Short history of the Internet and the Web

The Internet developed from early packet-switched networks. Packet switching breaks data into smaller packets that can travel through networks and be reassembled at the destination. ARPANET is commonly discussed as an important early network, while the later adoption of TCP/IP allowed independent networks to communicate using a common protocol family.

The Web came later. Tim Berners-Lee proposed a system in which documents could be identified by addresses, transferred using a standard protocol and connected using hyperlinks. The key ideas became **URL addressing, HTTP communication and HTML documents**. Browsers then made these linked documents practical for ordinary users.

Do not mix the two histories in an answer. Internet history is about interconnected networks; Web history is about the later linked-document system built on top of those networks.

# 2. Protocols governing the Web

A **protocol** is an agreed set of rules that tells communicating systems how information should be formatted, transmitted, received and interpreted. Web communication works because several protocols cooperate.

At the application level, **HTTP** carries requests and responses between clients and web servers. **HTTPS** is HTTP protected by TLS, adding encryption, server authentication and integrity protection. **DNS** converts domain names into IP addresses. Beneath them, **TCP** commonly provides reliable ordered transport, while **IP** handles logical addressing and routing between networks.

A simple browser-to-server flow is:

```
User enters URL
      ↓
Browser resolves domain name
      ↓
Connection is established with server
      ↓
Browser sends HTTP request
      ↓
Server sends HTTP response
      ↓
Browser interprets HTML/CSS and renders the page
```

# 3. Types of websites, web applications and web projects

A **static website** mainly serves pre-written resources such as HTML, CSS and images. The server normally returns the stored resource largely as it exists. Static does not necessarily mean non-interactive: client-side code can still make a statically served page interactive.

A **dynamic website** generates or selects content according to request data, database data, authentication state or application logic. An online store, for example, may display a different cart for every logged-in user.

A **web application** emphasizes interaction and data processing rather than only publishing information. Email clients, banking portals, dashboards and collaborative editors are typical examples. The boundary between website and web application is not absolute, but an application usually has more user-specific state and functionality.

A **web project** means the complete development effort, not only the final pages. It typically moves through requirements, content planning, design, implementation, testing, deployment and maintenance. This is useful in theory answers because it shows that web development is an engineering process rather than simply writing HTML.

# 4. Basic Web architecture

The fundamental model is **client-server architecture**. The **client**, usually a browser, requests a resource or service. The **server** receives the request, performs the required work and returns a response.

A dynamic system often has more layers:

```
Browser / Client
       ↓ HTTP request
Web server / Application server
       ↓
Business logic
       ↓
Database or other services
       ↑
HTTP response
       ↑
Browser renders result
```

The **frontend** is the part experienced by the user, usually HTML for structure and CSS for presentation. The **backend** handles server-side logic, authentication, database access and operations that should not be performed entirely in the browser. A **database** stores persistent application data.

# 5. URL and its anatomy

A **URL (Uniform Resource Locator)** identifies where a Web resource is located and how it should be accessed. Consider:

```
https://www.example.com:443/products/phone?id=27#reviews
```

Here `https` is the **scheme**, `www.example.com` is the **host/domain**, `443` is an optional **port**, `/products/phone` is the **path**, `?id=27` is the **query string**, and `#reviews` is the **fragment**. The fragment normally identifies a location or state within the retrieved document and is not ordinarily sent to the server as part of the HTTP request.

The URL matters because links, navigation, APIs and form submissions all require a consistent addressing system. A domain is only one part of a complete URL.

# 6. HTTP: why it is needed

**HTTP (Hypertext Transfer Protocol)** is the application-layer protocol used for communication between Web clients and servers. Its basic model is **request → response**. HTTP defines a standard message format so independently developed browsers and servers can understand each other.

HTTP is described as **stateless** because each request is conceptually independent. The protocol does not automatically remember that two requests came from the same logged-in user. Applications create continuity using cookies, sessions or tokens. Statelessness therefore does not mean that websites cannot remember users.

## 6.1 HTTP request format

An HTTP request contains a request line, headers, a blank line and, when necessary, a body.

```
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 24

username=student&mode=login
```

The request line gives the **method**, requested target and HTTP version. `GET` is commonly used to retrieve resources, while `POST` is commonly used to submit data for processing. Headers carry metadata; the body carries request data when required.

## 6.2 HTTP response format

A response contains a status line, headers, a blank line and usually a body.

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1250

<html>...</html>
```

Status codes summarize the result. `2xx` indicates success, `3xx` commonly indicates redirection, `4xx` indicates a client-side request problem and `5xx` indicates a server-side failure. Examples worth remembering are **200 OK**, **404 Not Found** and **500 Internal Server Error**.

# 7. Persistent and non-persistent connections

With a **non-persistent connection**, a separate transport connection is used for each requested resource. Repeated connection setup adds overhead when a page contains many resources.

With a **persistent connection**, one connection can be reused for multiple request-response exchanges. This reduces repeated connection-establishment overhead. The exam distinction is therefore **separate connection per exchange versus reusable connection for multiple exchanges**.

# 8. Web caching

A **cache** stores a previously obtained copy of a resource so that a later request may be served without downloading the entire resource again. Browsers maintain local caches, and intermediary systems can also cache content.

Caching reduces response time, bandwidth use and server load. The challenge is **freshness**: a cached copy may become outdated. HTTP provides cache-control and validation mechanisms so clients can decide whether a stored copy can be reused or should be checked with the origin server.

# 9. Proxy servers

A **proxy server** is an intermediary between a client and another server. Instead of contacting the destination directly, a client can send its request through a proxy. Proxies can provide caching, filtering, policy enforcement and access control.

A **forward proxy** primarily represents clients, while a **reverse proxy** stands in front of one or more servers. The syllabus only requires the proxy concept, so definition, intermediary role and common uses are the important parts.

# 10. Web server and Web browser

A **web server** is software that accepts HTTP requests and returns HTTP responses. It may serve static files directly or pass requests to application logic that generates dynamic results.

A **web browser** is the client application that requests resources and presents them to the user. It interprets HTML, applies CSS, loads referenced resources and manages navigation, caching and other browser features.

Do not define the browser as “the Internet.” The browser is only an application that uses Internet services. Similarly, “web server” often refers to a software role, not necessarily to one physical machine.

# 11. Internet standards

The Web depends on standards so that independently built browsers, servers and pages remain interoperable. Organizations such as the **IETF** develop many Internet protocol standards, while standards bodies and communities define Web technologies such as HTML and CSS.

For the midsem, the important idea is **interoperability**: standards prevent the Web from fragmenting into mutually incompatible vendor-specific systems.

# 12. TCP/IP protocol suite

The **TCP/IP suite** is the protocol family underlying Internet communication. A useful syllabus-level model has four layers.

- **Application layer:** protocols used by applications, such as HTTP and DNS.
- **Transport layer:** end-to-end communication between processes. TCP provides reliable ordered delivery; UDP provides a lighter connectionless service.
- **Internet layer:** IP provides addressing and routing between networks.
- **Link/network-access layer:** transfers frames over the local network technology, such as Ethernet or Wi-Fi.

When a browser sends an HTTP request, the application data moves down the stack, receives the information required by lower layers, travels through the network and is processed upward at the destination.

# 13. MIME

**MIME (Multipurpose Internet Mail Extensions)** provides standardized media types that describe what kind of content is being transmitted. HTTP uses these content types so a browser knows how to interpret a response.

```
text/html        → HTML document
text/css         → CSS stylesheet
image/png        → PNG image
application/json → JSON data
video/mp4        → MP4 video
```

The server commonly provides the media type through the `Content-Type` header. MIME therefore matters on the Web because a client must know whether received bytes represent HTML, an image, a video or some other format.

# 14. Issues in Web development and Cyber Laws

Web development involves more than visual appearance. **Compatibility** matters because browsers and devices differ. **Responsive design** matters because screen sizes vary. **Performance** matters because large files and excessive requests increase load time. **Accessibility** helps users with different abilities navigate and understand a site.

Security is equally important. User input should not be blindly trusted, sensitive information should be protected in transit, and authentication and authorization must be designed carefully. Maintainability also matters as projects grow; clear structure and separation of concerns make future changes safer and easier.

The syllabus also names **Cyber Laws**. At a conceptual level, cyber laws govern legal issues involving electronic records, online transactions, privacy, unauthorized access, cybercrime, intellectual property and misuse of digital systems. If your lecturer has prescribed specific statutes or cases, use those classroom notes as the authority because the syllabus itself only names the topic broadly.

---

# 15. HTML: structure and meaning of a Web page

**HTML (HyperText Markup Language)** describes the structure and meaning of Web content. It is called a *markup language* because special markup identifies headings, paragraphs, links, images, forms and other parts of a document. HTML should primarily describe what content *is*; CSS is used to control how that content looks.

An HTML **element** usually consists of an opening tag, content and a closing tag:

```html
<p>This is a paragraph.</p>
```

Some elements are **void elements** and do not wrap content, such as `<img>` and 
``. Elements can also have **attributes**, which provide additional information:

```html
<a href="https://example.com">Visit Example</a>
```

Here `a` is the element and `href` is an attribute that specifies the link destination.

# 16. Basic HTML document structure

A valid document follows a recognizable skeleton:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Page</title>
</head>
<body>
    <h1>Hello</h1>
    <p>This is the visible page content.</p>
</body>
</html>
```

`<!DOCTYPE html>` tells the browser to use modern HTML rules. `<html>` is the root element. `<head>` contains metadata and resource references that are not normally shown as the main page content. `<body>` contains the visible document.

HTML comments use the form `<!-- comment -->`. Comments are useful for developer notes, but they are still delivered with the page source and should never contain confidential information.

# 17. Important elements inside the document head

The `<title>` element sets the document title shown in the browser tab and used in browser history and related contexts.

The `<meta>` element stores metadata. Common examples declare the character encoding and viewport behavior:

```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

The `<link>` element connects the document to an external resource, most commonly a stylesheet:

```html
<link rel="stylesheet" href="styles.css">
```

The `<style>` element contains CSS rules written inside the current HTML document. The syllabus also lists the script element; its role is to include client-side script code or reference an external script resource. JavaScript itself is outside your stated midsem scope.

The `<base>` element sets a base URL for resolving relative URLs:

```html
<base href="https://example.com/docs/">
```

After this, a relative link such as `chapter1.html` is resolved relative to that base address. Because it affects many links at once, the base element should be used carefully.

# 18. Headings, paragraphs and line breaks

HTML provides six heading levels, `<h1>` through `<h6>`. These represent a **hierarchy**, not merely six font sizes. `<h1>` normally represents the main heading and lower levels organize subsections.

Paragraphs use `<p>`. Browsers collapse ordinary whitespace, so pressing Enter several times in the source does not create equivalent visual gaps. When a line break is semantically required within the same block, 
`` can be used. Separate paragraphs should normally use separate `<p>` elements rather than repeated line breaks.

# 19. Text formatting

HTML provides elements that convey emphasis and presentation. `<strong>` expresses strong importance and is normally rendered in bold, while `<em>` expresses emphasis and is normally italicized. `<b>` and `<i>` can produce conventional bold or italic presentation without exactly the same semantic meaning.

Other useful elements include `<mark>` for highlighted text, `<small>` for secondary text, `<sub>` for subscript and `<sup>` for superscript. In an exam answer, explaining the role of the tag is stronger than only listing tag names.

# 20. Anchors and hyperlinks

The anchor element creates hyperlinks:

```html
<a href="https://example.com">Open Example</a>
```

The `href` attribute can point to an absolute URL, a relative resource or a fragment inside the same document. For example:

```html
<a href="#contact">Jump to Contact</a>
<section id="contact">Contact information</section>
```

Hyperlinks are central to the idea of **hypertext** because they connect documents and sections rather than leaving each page isolated.

# 21. Images and video

Images are embedded using `<img>`:

```html
<img src="campus.jpg" alt="Main university building" width="600">
```

`src` identifies the image file. `alt` provides a text alternative and is important for accessibility and for cases where the image cannot be shown. Dimensions can be supplied in HTML, although responsive presentation is commonly controlled with CSS.

Video can be embedded with `<video>`:

```html
<video controls width="640">
    <source src="lecture.mp4" type="video/mp4">
    Your browser does not support the video element.
</video>
```

The `controls` attribute asks the browser to provide playback controls. `<source>` allows the media file and its MIME type to be specified.

# 22. HTML lists

An **unordered list** uses `<ul>` when item order is not important. An **ordered list** uses `<ol>` when sequence matters. Both use `<li>` for each item.

```html
<ul>
    <li>HTML</li>
    <li>CSS</li>
</ul>

<ol>
    <li>Plan the page</li>
    <li>Write the markup</li>
    <li>Apply styles</li>
</ol>
```

HTML also supports description lists through `<dl>`, `<dt>` and `<dd>`, which are useful for term-definition pairs. The important concept is that HTML provides the structure of the list; CSS can later alter markers and appearance.

# 23. Frames

The syllabus includes **frames**, historically used to divide a browser window into independently loaded documents. Traditional frame-based layouts are obsolete in modern HTML5, but the concept may still be examined because it remains in the syllabus.

The modern `<iframe>` element embeds another document inside the current page:

```html
<iframe src="help.html" title="Help document"></iframe>
```

For the exam, know the basic idea of displaying another document within a region of the current browsing context. Modern page layout should be handled by CSS rather than old-style frames.

# 24. HTML forms

A **form** collects user input and submits it for processing. The `<form>` element groups controls and defines where and how data is sent.

```html
<form action="/register" method="post">
    <label for="name">Name</label>
    <input id="name" name="name" type="text" required>

    <label for="email">Email</label>
    <input id="email" name="email" type="email">

    <button type="submit">Register</button>
</form>
```

The `action` attribute identifies the destination. `method="get"` usually places form fields in the URL query string and is suitable for retrieval-oriented operations such as search. `method="post"` places submitted data in the request body and is commonly used when sending or changing data. POST is not automatically encrypted; HTTPS provides protection in transit.

The `name` attribute matters because it becomes the field name in the submitted data. `<label>` improves usability and accessibility by identifying the purpose of an input. Common controls include text inputs, password inputs, radio buttons, checkboxes, file inputs, `<textarea>`, `<select>` and `<button>`.

A useful exam pattern is to be able to write a complete small form from memory and explain the difference between `action`, `method`, `name`, `id` and `type` rather than memorizing isolated tags.

---

# 25. CSS: separating presentation from structure

**CSS (Cascading Style Sheets)** controls the presentation of HTML. HTML should primarily describe document structure and meaning; CSS specifies colors, spacing, typography, layout and visual effects. Separating the two improves maintainability because one stylesheet can control the appearance of many elements or pages.

A CSS rule contains a **selector** and a **declaration block**:

```css
p {
    color: navy;
    line-height: 1.6;
}
```

The selector `p` chooses the elements. `color` and `line-height` are properties, while `navy` and `1.6` are their values.

# 26. Inline, internal and external CSS

**Inline CSS** is written directly inside an element's `style` attribute:

```html
<p style="color: red;">Important</p>
```

It affects that specific element. It can be useful for isolated cases, but excessive inline styling mixes structure and presentation and becomes difficult to maintain.

**Internal CSS** is written inside a `<style>` block in the document head:

```html
<style>
    p { color: red; }
</style>
```

It applies to that document and can be convenient for a single self-contained page.

**External CSS** is kept in a separate `.css` file and linked from HTML:

```html
<link rel="stylesheet" href="styles.css">
```

External CSS is normally best for multi-page sites because it promotes reuse and cleaner HTML. The key exam distinction is **where the rules are stored and how widely they can be reused**.

# 27. CSS selectors

Selectors determine which elements receive a style rule. The foundational selectors are:

```css
p { }             /* element selector */
.note { }         /* class selector */
#header { }       /* id selector */
* { }             /* universal selector */
```

A class can be reused on many elements, while an `id` is intended to identify one element within a document. Several selectors can share the same rule by grouping them:

```css
h1, h2, h3 {
    font-family: sans-serif;
}
```

Selectors can become more precise by expressing relationships, attributes and states. Those forms become especially important when understanding combinators, pseudo-classes and specificity.

# 28. The cascade and specificity

The word **cascading** means that several style rules may apply to the same element. The browser therefore needs a way to decide which declaration wins. For ordinary author styles, a useful specificity order is:

**inline style > ID selector > class / attribute / pseudo-class > element / pseudo-element**.

For example:

```css
p { color: black; }
.notice { color: blue; }
#warning { color: red; }
```

An element matching all three normally receives the `#warning` color because the ID selector is more specific. If two competing rules have equal specificity, the one appearing later normally wins.

A common mistake is to say “the last rule always wins.” Source order matters only after stronger cascade factors and specificity have been considered.

# 29. Colors and backgrounds

CSS can represent colors using names, hexadecimal values, `rgb()` and `rgba()` among other formats.

```css
.title { color: #1f3a5f; }
.panel { background-color: rgb(245, 245, 245); }
.overlay { background-color: rgba(0, 0, 0, 0.4); }
```

Backgrounds can also use images:

```css
.hero {
    background-image: url("banner.jpg");
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;
}
```

`background-attachment` controls whether a background scrolls with the document or remains fixed relative to the viewport in supported contexts. CSS also provides the shorthand `background` property for setting several background values together. Remember that shorthand properties may reset omitted sub-properties to their defaults.

# 30. Borders, margin and padding: the box model

Every ordinary element can be understood using the **CSS box model**. At the center is the content area. Around it comes **padding**, then **border**, then **margin**.

**Padding** is space inside the border, between the content and border. **Border** surrounds the content and padding. **Margin** is space outside the border and separates the element from neighboring elements.

```css
.card {
    width: 300px;
    padding: 20px;
    border: 2px solid #333;
    margin: 16px;
}
```

With the default `box-sizing: content-box`, the declared width describes the content width, so padding and borders add to the final visible size. With `box-sizing: border-box`, the declared width includes content, padding and border. This distinction is useful when a numerical or layout question asks why an element appears wider than expected.

Borders can vary in width, style, color and radius. Margin and padding support individual sides as well as shorthand forms such as `margin: 10px 20px`, which means 10 px vertically and 20 px horizontally.

# 31. Fonts and text presentation

Typography strongly affects readability. Important font properties include `font-family`, `font-size`, `font-weight`, `font-style` and `line-height`.

```css
body {
    font-family: Arial, sans-serif;
    font-size: 16px;
    line-height: 1.6;
}
```

The browser tries the listed fonts in order until it finds one available. A generic family such as `sans-serif` provides a fallback.

Text properties include `text-align`, `text-decoration`, `text-transform`, `letter-spacing`, `word-spacing` and `text-shadow`. Font properties describe the typeface and its metrics, while text properties describe how the text is arranged or decorated.

# 32. Styling links, icons, lists and tables

Links can be styled like other elements, but their interaction states are often handled with pseudo-classes such as `:link`, `:visited`, `:hover` and `:active`. A good style should keep links recognizable rather than removing every visual clue that they are interactive.

Icons may come from images, SVGs, icon fonts or libraries. From a CSS perspective, the key idea is that icons can often be sized, colored and aligned consistently with surrounding content depending on how they are implemented.

Lists can change their marker type, marker position or use images through `list-style` properties. Tables can be styled using borders, padding, alignment and alternating row effects. `border-collapse: collapse` is commonly used so adjacent table borders combine rather than appearing doubled.

# 33. The `display` property

`display` controls how an element participates in layout. A **block** element starts on a new line and generally uses the available horizontal space. An **inline** element flows within surrounding text and does not behave like a normal rectangular block for width and height. An **inline-block** element flows inline while still accepting width, height, padding and margin like a box.

`display: none` removes an element from layout entirely. This differs from `visibility: hidden`, which hides the element visually while normally preserving its layout space.

Modern CSS also provides `flex` and `grid`. They are not explicitly named in the supplied syllabus, so treat them as part of “Website Layout” only if your lecturer has actually covered them.

# 34. CSS positioning

The `position` property changes how offsets such as `top`, `right`, `bottom` and `left` are interpreted.

**`static`** is the normal default. Offset properties do not move a statically positioned element.

**`relative`** keeps the element in normal flow but allows it to be visually offset relative to its original position. Its original layout space remains reserved.

**`absolute`** removes the element from normal flow and positions it relative to an appropriate containing block, commonly the nearest positioned ancestor.

**`fixed`** positions the element relative to the viewport, so it remains in place during scrolling.

**`sticky`** behaves normally until a scroll threshold is reached, after which it sticks within its scrolling context.

For an exam answer, do not only list these values. Explain whether the element remains in normal flow and what reference point is used for positioning.

# 35. Overflow

When content is larger than an element's box, `overflow` determines what happens. `visible` allows content to extend outside, `hidden` clips it, `scroll` provides scrolling, and `auto` adds scrolling only when needed.

```css
.panel {
    width: 300px;
    height: 150px;
    overflow: auto;
}
```

Horizontal and vertical behavior can also be controlled separately with `overflow-x` and `overflow-y`.

# 36. Float and inline-block

`float` was originally intended to let surrounding text wrap around content such as images. It later became a common older layout technique, but floated layouts need careful handling because floats affect normal document flow.

```css
img.thumb {
    float: left;
    margin-right: 12px;
}
```

`inline-block` was another common pre-Flexbox layout technique. It allows several box-like elements to appear on the same line while still accepting width and height. For the midsem, know their behavior and historical layout role; neither should be confused with absolute positioning.

# 37. Horizontal and vertical alignment

Horizontal alignment depends on what is being aligned. Text inside a block can be aligned using `text-align`. A block with a defined width can often be centered horizontally using automatic side margins:

```css
.container {
    width: 80%;
    margin: 0 auto;
}
```

Vertical alignment is more context-dependent. `vertical-align` applies to inline-level and table-cell contexts; it is not a universal “center anything vertically” property. Modern layouts frequently use Flexbox or Grid for robust centering, but if those techniques were not covered in class, focus on the alignment mechanisms your lecturer actually taught.

# 38. CSS combinators

Combinators describe relationships between elements and allow selectors to express document structure.

```css
article p { }      /* descendant: any p inside article */
article > p { }    /* child: p directly inside article */
h2 + p { }         /* adjacent sibling: first p immediately after h2 */
h2 ~ p { }         /* general sibling: later p siblings of h2 */
```

The distinction between **descendant** and **child** selectors is particularly important. A descendant may be nested at any depth; a child must be directly inside the selected parent. Sibling combinators work between elements that share the same parent.

# 39. Pseudo-classes and pseudo-elements

A **pseudo-class** selects an element according to a state, position or condition without requiring another class in the HTML.

```css
a:hover { color: red; }
input:focus { outline: 2px solid blue; }
li:first-child { font-weight: bold; }
```

A **pseudo-element** styles a conceptual part of an element or inserts generated presentation content.

```css
p::first-line { font-weight: bold; }
.note::before { content: "Note: "; }
```

The easiest distinction is: **pseudo-class = state or condition of an element; pseudo-element = part or generated portion of an element**.

# 40. Attribute selectors

Attribute selectors choose elements according to the presence or value of HTML attributes.

```css
input[type="email"] { }
a[target="_blank"] { }
[class^="btn-"] { }
```

They are useful when the HTML already contains meaningful attributes and an extra class would be unnecessary. Operators can also match prefixes, suffixes and substrings, so attribute selectors can be very precise.

# 41. Opacity and transparency

The `opacity` property changes the transparency of the entire rendered element, including its children:

```css
.card {
    opacity: 0.6;
}
```

If only a background should be translucent while the text remains fully opaque, an alpha color such as `rgba()` is usually more appropriate. This is a useful exam distinction: **element opacity affects the whole element subtree, whereas alpha in a particular color affects only that color**.

# 42. Navigation bars

A navigation bar is normally built from links in HTML and then arranged and styled with CSS.

```html
<nav class="nav">
    <a href="/">Home</a>
    <a href="/about">About</a>
    <a href="/contact">Contact</a>
</nav>
```

```css
.nav a {
    display: inline-block;
    padding: 12px 16px;
    text-decoration: none;
}

.nav a:hover {
    background: #eee;
}
```

The important concept is not one particular navbar design. HTML provides the navigation links; CSS controls their spacing, arrangement, colors and interaction states.

# 43. Dropdowns

A simple CSS dropdown contains a trigger and hidden content. The content becomes visible when the parent reaches a chosen state such as `:hover`.

```css
.dropdown-content {
    display: none;
    position: absolute;
}

.dropdown:hover .dropdown-content {
    display: block;
}
```

This one example connects several syllabus topics: descendant selectors, pseudo-classes, `display` and absolute positioning. In real applications keyboard accessibility may require additional handling, but the basic CSS mechanism is the important syllabus idea.

# 44. Image galleries

An image gallery is a repeated layout of image items with consistent sizing, spacing and captions. CSS can control thumbnail dimensions, borders, hover effects and wrapping. A common responsive rule is `max-width: 100%`, which helps prevent an image from overflowing its container.

The topic also demonstrates why reusable classes matter. Instead of styling every image separately, one class can be applied to all repeated gallery items, keeping the presentation consistent and easier to maintain.

# 45. Styling forms with CSS

CSS can make forms easier to understand by creating consistent spacing, label alignment, input sizing and visible focus states.

```css
input,
select,
textarea {
    width: 100%;
    padding: 10px;
    border: 1px solid #bbb;
    border-radius: 4px;
}

input:focus {
    border-color: #3366cc;
}
```

Form styling is not only decorative. Good styling should make fields easy to scan, make focus obvious for keyboard users and keep related labels and controls visually connected.

# 46. CSS counters

CSS counters allow automatic numbering to be generated by styles instead of manually typing numbers into the HTML. A counter is reset on a container, incremented on repeated elements and displayed using generated content.

```css
body {
    counter-reset: section;
}

h2::before {
    counter-increment: section;
    content: counter(section) ". ";
}
```

This example combines counters with the `::before` pseudo-element. If the heading order changes, the displayed numbering updates automatically.

# 47. Website layout

A website layout organizes major regions such as the header, navigation area, main content, sidebar and footer. The important concept is **clear structural organization**, not memorizing one fixed template.

```
+---------------------------+
|          Header           |
+---------------------------+
|        Navigation         |
+-------------+-------------+
| Main        | Sidebar     |
| Content     |             |
+-------------+-------------+
|          Footer           |
+---------------------------+
```

Older layouts often used floats and inline-block. Modern sites commonly use Flexbox or Grid. Regardless of technique, layout should keep content understandable and adaptable across screen sizes.

# 48. CSS units

CSS uses both **absolute** and **relative** units. `px` is a common CSS pixel unit. Relative units adapt to another measurement: `%` depends on a contextual reference, `em` depends on an element's font-size context, `rem` depends on the root element's font size, while `vw` and `vh` depend on viewport dimensions.

The important exam idea is why relative units are useful: they can make typography and layouts adapt to their context. Do not memorize the idea that relative units are always superior; each unit has appropriate uses.

# 49. Text effects

CSS provides effects through properties such as `text-shadow`, `text-overflow`, `overflow-wrap` and related controls. A common truncation pattern is:

```css
.title {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}
```

The three declarations work together: prevent wrapping, hide the extra text and display an ellipsis. Understanding the combined behavior is more useful than memorizing `text-overflow` alone.

# 50. CSS animations

CSS animations allow property values to change over time without requiring script code for the basic animation. First define keyframes, then apply them to an element.

```css
@keyframes fadeIn {
    from { opacity: 0; }
    to   { opacity: 1; }
}

.notice {
    animation: fadeIn 0.6s ease-out;
}
```

Important animation settings include the name, duration, timing function, delay, iteration count and direction. A related concept is a **transition**, which smoothly changes a property when its value changes, for example during `:hover`. An animation can run through defined keyframes independently; a transition normally responds to a change between property values.

# 51. Tooltips

A tooltip displays a small explanatory label when the user interacts with an element. A simple CSS tooltip can keep the label hidden initially and reveal it when the parent is hovered.

```css
.tooltip-text {
    visibility: hidden;
    position: absolute;
}

.tooltip:hover .tooltip-text {
    visibility: visible;
}
```

The parent is often positioned relatively so the absolutely positioned tooltip can be placed in relation to it. This topic is easier to remember when connected to positioning, pseudo-classes and visibility rather than learned as an isolated recipe.

# 52. Multiple columns

CSS multi-column layout can flow text into newspaper-style columns.

```css
.article {
    column-count: 3;
    column-gap: 24px;
}
```

Instead of fixing the number of columns, `column-width` can request an approximate preferred width and let the browser determine how many columns fit. Multi-column layout is intended for flowing content and is conceptually different from building independent page regions using a grid-like layout.

---

# 53. High-value distinctions for the midsem

These distinctions are worth being able to explain in your own words because they connect several syllabus topics and prevent common mistakes.

- **Internet vs WWW:** global networking infrastructure versus a hyperlinked information and application system running over it.
- **Static vs dynamic website:** stored resources served largely as-is versus content selected or generated through application logic and data.
- **Browser vs web server:** client that requests and renders resources versus server that receives requests and returns responses.
- **URL vs domain:** complete resource address versus only the host-name portion.
- **HTTP request vs response:** client-to-server message versus server-to-client result.
- **Persistent vs non-persistent connection:** reuse one connection for multiple exchanges versus creating separate connections.
- **Margin vs padding:** space outside the border versus space inside the border.
- **Block vs inline vs inline-block:** different participation in line layout and different width/height behavior.
- **Relative vs absolute positioning:** relative stays in normal flow and offsets from its original position; absolute leaves normal flow and uses a containing block as its reference.
- **Pseudo-class vs pseudo-element:** state or condition of an element versus a conceptual part or generated portion.
- **Inline vs internal vs external CSS:** style on one element, rules inside one document, or rules in a reusable stylesheet file.
- **Specificity vs source order:** selector strength is considered before later source order decides between equally strong competing rules.

# 54. Final revision method

For **The Web**, be able to explain the browser → network → server request/response flow and define Internet, WWW, URL, HTTP, browser, server, caching, proxy, TCP/IP and MIME without looking at the notes.

For **HTML**, be able to write a valid document skeleton from memory and construct a small page containing headings, paragraphs, links, images, lists and a form. Focus on what each tag contributes to structure rather than memorizing a catalogue of tags.

For **CSS**, be able to style that page while explaining the box model, selector types, specificity, display, positioning, overflow, combinators, pseudo-classes and pseudo-elements. Those concepts make many of the later topics—dropdowns, tooltips, navigation bars and galleries—much easier to reconstruct.

Do not use the final revision only for rereading. A stronger check is whether you can answer prompts such as **“How does a browser obtain a page?”**, **“Write and explain an HTML form,”** or **“Why did this CSS rule win?”** from memory. If you can reconstruct the mechanism, the smaller definitions are much easier to retain.