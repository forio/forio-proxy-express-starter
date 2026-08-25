# Forio Proxy Starter

Starter code for running an Express server using an Epicenter proxy.

## Configuration

`npm run deploy` uploads and resets the proxy, but it does not configure the following
Epicenter settings. Set them before the first deployment.

1. **Set the organization shared secret (organization owners only).**

   Open the organization that owns the project, then go to **Settings → General Settings**.
   Expand **More Options** and, under **Account API Key**, select **Set Shared Secret Key**.
   Enter a non-empty secret and select **Save** in the dialog.

   Only organization Owners may set this secret. Changing an existing shared secret can
   break other applications that rely on it.

2. **Enable the project's proxy server.**

   Open the project, then go to **Settings → General Settings** and expand **Advanced
   Settings**. Under **Proxy Settings**:

   - Set **Project Server** to **Proxy Server**.
   - Set **Proxy Model Filename** to `index.js`.
   - Select **Save Project Settings**.

   Use `index.js`, not `index.ctx2`: `index.js` is the server entry point, while `index.ctx2`
   supplies its runtime configuration. The deployment script copies both files from `src/`
   into the project's proxy directory.

Project-scoped proxies require a team organization, a team project, and an eligible plan or
an account specifically enabled by Forio. If the account is not enabled, the **Proxy
Settings** section may show this message instead of the server controls:

> Looking for proxy and/or Node.js server configuration? Your account is not yet configured
> to allow project-scoped models. Please contact support@forio.com to enable this feature.

Contact Forio before continuing if you see that message.

## Deployment

1. Copy `tools/config.example.json` to `tools/config.json` and fill in the
   required fields. Do not commit `config.json`.
2. `npm run deploy`

## Consumption

Once deployed, fetch against
`https://forio.com/proxy/<account>/<project>/<...path>`.

This starter comes with a handler for `GET /`. Requesting `GET /` will return a
simple message indicating that the server is running.

A handler for `POST /completions` would receive requests at
`https://forio.com/proxy/<account>/<project>/completions`, and so on.

### Example

```typescript
/**
 * Fetch against the proxy with the native `fetch` API.
 * Optional `normalizeFetchResponse` utility aligns `fetch` behavior
 * with other `epicenter-libs` requests. (see `tools/epicenter.ts`)
 */

import { normalizeFetchResponse } from './tools/epicenter';

const completion = await fetch(
  'https://forio.com/proxy/<account>/<project>/completions',
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${token}`,
    },
    body: JSON.stringify({
      prompt:
        "What's a good strategy for managing an employee who keeps mumbling about a missing stapler?",
    }),
  }
).then(normalizeFetchResponse);
```

## Common use cases

- Authenticate Epicenter with a privileged `secretKey` to perform privileged
  actions:

  ```javascript
  authAdapter
    .login({ secretKey: epicenter.proxyConfig().apiSharedSecret })
    .then((session) => session /* AccountWhoAmI */);
  ```

- Hold secrets on the server and route requests to the proxy.

## Development

### Restarting the proxy

Upon redeploying proxy code, you'll want to restart the process.
`npm run deploy` stops the proxy as part of its flow. You can also
`npm run stop` to do the same. The proxy will restart automatically when a
request is routed to it.

### Logging

Define a `minimumLogLevel` in `src/index.ctx2` to record logs during the proxy
process.

Options for `minimumLogLevel` are, from most to least verbose:

- TRACE
- DEBUG
- INFO
- WARN
- ERROR
- FATAL
- OFF

Use `epicenter.log` in proxy code to log messages:

```javascript
/* First argument categorizes the log (see above),
 * second is the message. Match the message to its severity
 * and be sure minimumLogLevel is set to the appropriate level.
 */
epicenter.log('INFO', `Listening on port ${port}`);
epicenter.log('ERROR', `Error: ${JSON.stringify(err)}`);
```

Run `npm run log` to fetch the proxy log and save it locally to `/logs`.
