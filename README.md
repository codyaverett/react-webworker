# `<WebWorker>`

[![npm version](https://img.shields.io/npm/v/react-webworker.svg)](https://www.npmjs.com/package/react-webworker)
[![npm downloads](https://img.shields.io/npm/dm/react-webworker.svg)](https://www.npmjs.com/package/react-webworker)
[![ISC license](https://img.shields.io/npm/l/react-webworker.svg)](https://opensource.org/licenses/ISC)
[![minified size](https://img.shields.io/bundlephobia/min/react-webworker.svg)](https://bundlephobia.com/result?p=react-webworker)
[![GitHub issues](https://img.shields.io/github/issues/ghengeveld/react-webworker.svg)](https://github.com/ghengeveld/react-webworker/issues)
[![GitHub PRs](https://img.shields.io/github/issues-pr/ghengeveld/react-webworker.svg)](https://github.com/ghengeveld/react-webworker/pulls)

React component for easy communication with a Web Worker. Leverages the Render Props pattern for ultimate flexibility as
well as the new Context API for ease of use. Just specify the public url to your Web Worker and you'll get access to
any messages or errors it sends, as well as the `postMessage` handler.

- Zero dependencies
- Choose between Render Props and Context-based helper components
- Provides timestamped `messages` and `errors`
- Provides easy access to the last message `data` and last `error`
- Provides `postMessage` to send messages to the Web Worker
- Provides `updatedAt` and `lastPostAt` metadata
- Accepts `parser` and `serializer` for automatic message (de)serialization
- Accepts `onMessage` and `onError` callbacks

> This package was modeled after [`<Async>`](https://github.com/ghengeveld/react-async) which helps you deal with Promises in React.

## Install

```
npm install --save react-webworker
```

## Usage

Using render props for ultimate flexibility:

```js
import WebWorker from "react-webworker"

const MyComponent = () => (
  <WebWorker url="/worker.js">
    {({ data, error, postMessage }) => {
      if (error) return `Something went wrong: ${error.message}`
      if (data)
        return (
          <div>
            <strong>Received some data:</strong>
            <pre>{JSON.stringify(data, null, 2)}</pre>
          </div>
        )
      return <button onClick={() => postMessage("hello")}>Hello</button>
    }}
  </WebWorker>
)
```

Using helper components (don't have to be direct children) for ease of use:

```js
import WebWorker from "react-webworker"

const MyComponent = () => (
  <WebWorker url="/worker.js">
    <WebWorker.Pending>
      {({ postMessage }) => <button onClick={() => postMessage("hello")}>Hello</button>}
    </WebWorker.Pending>
    <WebWorker.Data>
      {data => (
        <div>
          <strong>Received some data:</strong>
          <pre>{JSON.stringify(data, null, 2)}</pre>
        </div>
      )}
    </WebWorker.Data>
    <WebWorker.Error>{error => `Something went wrong: ${error.message}`}</WebWorker.Error>
  </WebWorker>
)
```

### Props

`<WebWorker>` takes the following properties:

- `url` {string} (required) Public url to the Web Worker file (or path relative to the root of your domain)
- `options` {Object} Options passed to the Worker constructor
- `parser` {Function} Transforms incoming message data (not errors)
- `serializer` {Function} Transforms `postMessage` payload before sending
- `onMessage` {Function} Callback function invoked when a message is received, passing message data as argument
- `onError` {Function} Callback function invoked when an error is received, passing error object as argument

> `url` and `options` are evaluated at mount time, so they must be defined immediately and won't respond to changes.

### Render props

`<WebWorker>` provides the following render props:

- `messages` {Array} list of received messages ({ data, date }), in chronological order
- `errors` {Array} list of received errors ({ error, date }), in chronological order
- `data` {any} last received message data, maintained when an error is received
- `error` {Error} last received error, cleared when new message arrives
- `updatedAt` {Date} when the last message or error was received
- `postMessage` {Function} sends a message to the Web Worker

> Note: [it's recommended](https://nolanlawson.com/2016/02/29/high-performance-web-worker-messages/) to send and receive
> JSON strings instead of JS objects for improved performance. You can use the `parser` and `serializer` props to have
> `<WebWorker>` deal with this on the client side, but your Worker must still (de)serialize messages on its end.

## Examples

### Using `lastPostAt` to show a loading indicator

```js
import WebWorker from "react-webworker"

const MyComponent = () => (
  <WebWorker url="/worker.js">
    {({ data, error, postMessage, updatedAt, lastPostAt }) => (
      <div>
        {data && (
          <div>
            <strong>Received some data:</strong>
            <pre>{JSON.stringify(data, null, 2)}</pre>
          </div>
        )}
        <button onClick={() => postMessage("hello")} disabled={updatedAt < lastPostAt}>
          {updatedAt < lastPostAt ? "Loading..." : "Go"}
        </button>
      </div>
    )}
  </WebWorker>
)
```

### Passing options to the Worker

```js
import WebWorker from "react-webworker"

const MyComponent = () => (
  <WebWorker url="/worker.js" options={{ type: "module", credentials: "include" }}>
    ...
  </WebWorker>
)
```

### Using `parser` and `serializer` to automatically parse incoming messages and stringify outgoing messages

```js
import WebWorker from "react-webworker"

const MyComponent = () => (
  <WebWorker url="/worker.js" parser={JSON.parse} serializer={JSON.stringify}>
    {({ data, error, postMessage, updatedAt, lastPostAt }) => (
      <div>
        {data && (
          <div>
            <strong>Received some data:</strong>
            <pre>{JSON.stringify(data, null, 2)}</pre>
          </div>
        )}
        <button onClick={() => postMessage({ foo: "bar" })}>Send</button>
      </div>
    )}
  </WebWorker>
)
```

> Note: the Worker must still implement JSON (de)serialization on its own end.

## Helper components

`<WebWorker>` provides several helper components that make your JSX even more declarative.
They don't have to be direct children of `<WebWorker>` and you can use the same component several times.

### `<WebWorker.Data>`

Renders only when a message has been received.

#### Props

- `children` {Function|Node} Render function which receives last message data and props object or just a plain React node.

#### Examples

```js
<WebWorker.Data>{data => <pre>{JSON.stringify(data)}</pre>}</WebWorker.Data>
```

### `<WebWorker.Error>`

Renders only when an error has been received.

#### Props

- `children` {Function|Node} Render function which receives error and props object or just a plain React node.

#### Examples

```js
<WebWorker.Error>{error => `Unexpected error: ${error.message}`}</WebWorker.Error>
```

### `<WebWorker.Pending>`

Renders only when no message or error has been received yet. Enable `persist` to ignore errors.

#### Props

- `persist` {boolean} Show until we receive message data, even when an error is received.
- `children` {Function|Node} Function which receives props object or React node.
