# Methods and events

This section of SDK covers the topic related
to [apps communication](../../../platform/apps-communication/flow-definition.md).

## Calling methods

To call the Telegram Mini Apps methods, developer should use `postEvent` function:

```typescript
import { postEvent } from '@tma.js/sdk';

postEvent('web_app_setup_back_button', { is_visible: true });
```

This function automatically finds the correct way to send this event based on the current
environment features. For greater accuracy, it determines the current Telegram application type and
selects the appropriate flow.

## Listening to events

### `on` and `off`

To start working with events, developer could use `on` and `off` functions. Here is how basic
`on` function usage looks like:

```typescript
import { on } from '@tma.js/sdk';

// Start listening to "viewport_changed" event. Returned value
// is a function, which removes this event listener.
const removeListener = on('viewport_changed', payload => {
  console.log('Viewport changed:', payload);
});

// Remove this event listener.
removeListener();
```

To stop listening to events, developer could alternatively use `off` function:

```typescript
import { on, off, type MiniAppsEventListener } from '@tma.js/sdk';

const listener: MiniAppsEventListener<'viewport_changed'> = payload => {
  console.log('Viewport changed:', payload);
};

// Start listening to event.
on('viewport_changed', listener);

// Remove event listener.
off('viewport_changed', listener);
```

### `subscribe` and `unsubscribe`

To listen to all events sent from the native Telegram application, developer should utilize
such functions as `subscribe` and `unsubscribe`:

```typescript
import {
  subscribe,
  unsubscribe,
  type MiniAppsGlobalEventListener,
} from '@tma.js/sdk';

const listener: MiniAppsGlobalEventListener = (event, data) => {
  console.log('Received event', event, 'with data', data);
};

// Listen to all events.
subscribe(listener);

// Remove this listener.
unsubscribe(listener);
```

## Checking method support

`postEvent` function itself is not checking if specified method supported by current native Telegram
application. To do this, developer could use `supports` function which accepts Telegram Mini Apps
method name and current platform version:

```typescript
import { supports } from '@tma.js/sdk';

supports('web_app_trigger_haptic_feedback', '6.0'); // false
supports('web_app_trigger_haptic_feedback', '6.1'); // true
```

The `supports` function also allows checking if specified parameter in method parameters is
supported:

```typescript
import { supports } from '@tma.js/sdk';

supports('web_app_open_link', 'try_instant_view', '6.0'); // false
supports('web_app_open_link', 'try_instant_view', '6.7'); // true
```

::: tip
It is recommended to use this function before calling Telegram Mini Apps methods to prevent
applications from stalling and other unexpected behavior.
:::

### Creating safer `postEvent`

This package includes a function named `createPostEvent` that takes the current Telegram Mini Apps
version as input. It returns the `postEvent` function, which internally checks if the specified
method and parameters are supported. If they are not, the function will throw an error.

```typescript
import { createPostEvent } from '@tma.js/sdk';

const postEvent = createPostEvent('6.5');

// Will work fine.
postEvent('web_app_read_text_from_clipboard');

// Will throw an error.
postEvent('web_app_request_phone');
```

It is highly recommended to use this `postEvent` generator to ensure that method calls work as
expected.

## Debugging

Package supports enabling the debug mode, which leads to logging messages related to events
handling. To change debug mode, use `setDebug` function:

```typescript
import { setDebug } from '@tma.js/sdk';

setDebug(true);
```

## Target origin

If the package is being used in a browser environment (iframe), packages employs the
function `window.parent.postMessage`. This function requires specifying the target origin to ensure
events are only sent to trusted parent iframes. By default, the package
utilizes `https://web.telegram.org` as the origin. To enable event transmission to other origins,
developer should utilize the `setTargetOrigin` function:

```typescript
import { setTargetOrigin } from '@tma.js/sdk';

setTargetOrigin('https://myendpoint.org');
```

::: warning
It is strongly recommended not to override this value as long as it could lead to security issues.
Specify this value only for test purposes.
:::