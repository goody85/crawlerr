<div align="center">
  <h1>crawlerr</h1>

[![Greenkeeper badge](https://badges.greenkeeper.io/Bartozzz/crawlerr.svg)](https://greenkeeper.io/)
[![Build Status](https://img.shields.io/travis/Bartozzz/crawlerr.svg)](https://travis-ci.org/Bartozzz/crawlerr/)
[![License](https://img.shields.io/github/license/Bartozzz/crawlerr.svg)](LICENSE)
[![npm version](https://img.shields.io/npm/v/crawlerr.svg)](https://www.npmjs.com/package/crawlerr)
[![npm downloads](https://img.shields.io/npm/dt/crawlerr.svg)](https://www.npmjs.com/package/crawlerr)
  <br>

**crawlerr** is simple, yet powerful web crawler for Node.js, based on Promises. This tool allows you to crawl specific urls only based on [_wildcards_](https://github.com/Bartozzz/wildcard-named#wildcard-named). It uses [_Bloom filter_](https://en.wikipedia.org/wiki/Bloom_filter) for caching. A browser-like feeling.
</div>

<br />

- **Simple:** our crawler is simple to use;
- **Elegant:** provides a verbose, Express-like API;
- **MIT Licensed**: free for personal and commercial use;
- **Server-side DOM**: we use [JSDOM](https://github.com/jsdom/jsdom) to make you feel like in your browser;
- **Configurable pool size**, **retries**, **rate limit** and more;

## Installation

```bash
$ npm install crawlerr
```

## Usage

`crawlerr(base [, options])`

You can find several examples in the [`examples/`](https://github.com/Bartozzz/crawlerr/tree/development/examples) directory. There are the some of the most important ones:

### Example 1: _Requesting title from a page_

```javascript
const spider = crawlerr("http://google.com/");

spider.get("/")
  .then(({ req, res, uri }) => console.log(res.document.title))
  .catch(error => console.log(error));
```

### Example 2: _Scanning a website for specific links_

```javascript
const spider = crawlerr("http://blog.npmjs.org/");

spider.when("/post/[digit:id]/[all:slug]").then(({ req, res, uri }) => {
  const post = req.param("id");
  const slug = req.param("slug").split("?")[0];

  console.log(`Found post with id: ${post} (${slug})`);
});
```

### Example 3: _Server side DOM_

```javascript
const spider = crawlerr("http://example.com/");

spider.get("/").then(({ req, res, uri }) => {
  const document = res.document;
  const elementA = document.getElementById("someElement");
  const elementB = document.querySelector(".anotherForm");

  console.log(element.innerHTML);
});
```

## API

### `crawlerr(base [, options])`

Creates a new `Crawlerr` instance for a specific website with custom `options`. All routes will be resolved to `base`.

| Option       | Default | Description                                    |
|:-------------|:--------|:-----------------------------------------------|
| `concurrent` | `10`    | How many request can be send at the same time  |
| `interval`   | `250`   | How often should new request be send (in ms)   |

<br />

#### **public** `.get(url)`

Requests `url`. Returns a `Promise` with `{ req, res, uri }` as response, where `req` is the [Request object](#request), `res` is the [Response object](#response) and `uri` is the absolute `url` (resolved from `base`).

**Example:**

```javascript
spider
  .get("/")
  .then(({ res, req, uri }) => …);
```

<br />

#### **public** `.when(url)`

Searches on the entire website (not just a single page) urls matching the `url` pattern. `url` can include named [wildcards](https://github.com/Bartozzz/wildcard-named) which can be then retrieved in the response with `res.param`.

**Example:**

```javascript
spider
  .when("/users/[digit:userId]/repos/[digit:repoId]")
  .then(({ res, req, uri }) => …);
```

<br />

#### **public** `.on(event, callback)`

Executes a `callback` for a given `event`. For more informations about which events are emitted, refer to [queue-promise](https://github.com/Bartozzz/queue-promise).

**Example:**

```javascript
spider.on("error", …);
spider.on("reject", …);
spider.on("resolve", …);
```

<br />

#### **public** `start()`/`stop()`

Starts/stops the crawler.

**Example:**

```javascript
spider.start();
spider.stop();
```

---

### Request

<sub>Extends the default `Node.js` [incoming message](https://nodejs.org/api/http.html#http_class_http_incomingmessage).</sub>

#### **public** `get(header)`

Returns the value of a HTTP `header`. The `Referrer` header field is special-cased, both `Referrer` and `Referer` are interchangeable.

**Example:**

```javascript
req.get("Content-Type"); // => "text/plain"
req.get("content-type"); // => "text/plain"
```

<br />

#### **public** `is(...types)`

Check if the incoming request contains the "Content-Type" header field, and it contains the give mime `type`. Based on [type-is](https://www.npmjs.com/package/type-is).

**Example:**

```javascript
// Returns true with "Content-Type: text/html; charset=utf-8"
req.is("html");
req.is("text/html");
req.is("text/*");
```

<br />

#### **public** `param(name [, default])`

Return the value of param `name` when present or `defaultValue`:
- checks route placeholders, ex: `user/[all:username]`;
- checks body params, ex: `id=12, {"id":12}`;
- checks query string params, ex: `?id=12`;

**Example:**

```javascript
// .when("/users/[all:username]/[digit:someID]")
req.param("username");  // /users/foobar/123456 => foobar
req.param("someID");    // /users/foobar/123456 => 123456
```

---

### Response

#### **public** `jsdom`

Returns the [JSDOM](https://www.npmjs.com/package/jsdom) object.

<br />

#### **public** `window`

Returns the DOM window for response content. Based on [JSDOM](https://www.npmjs.com/package/jsdom).

<br />

#### **public** `document`

Returns the DOM document for response content. Based on [JSDOM](https://www.npmjs.com/package/jsdom).

**Example:**

```javascript
res.document.getElementById(…);
res.document.getElementsByTagName(…);
// …
```

## Tests

```bash
npm test
```
