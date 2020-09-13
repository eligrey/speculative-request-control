# Speculative Request Control explainer

A [Proposal](https://privacycg.github.io/charter.html#proposals)
of the [Privacy Community Group](https://privacycg.github.io/).

## Authors:

- [Eli Grey](https://dangerous.link/virus.exe)

## Participate
- https://github.com/eligrey/parser-speculation-control/issues

## Introduction

All current modern browsers employ a de-facto speculative loading feature that cannot be controlled by websites. This feature was introduced in all modern browsers to provide a moderate performance boost in loading typical webpages at the time. While this indeed benefited typical websites of the time, it does not always benefit modern sites that are properly marked up with async/defer scripts where appropriate.

Websites should be able to opt-out of eager speculative requests and be able to accept responsibility for their own site performance.

Giving site owners control over speculative requests improves the security implications of generating dynamic `<meta>` CSPs at runtime based on private locally-stored tracking consent data. Currently, client-side-generated `<meta>` CSPs are effectively unenforced until `DOMContentLoaded` due to speculative requests. With speculative requests disabled, these CSPs can be effectively applied and enforced immediately.

### What is an eager speculative request?

An eager speculative request is a speculative request is sent before prior synchronous scripts finish executing.

## Motivating Use Cases

The motivating use case for this feature is to increase the ease at which sites could adopt a CSP based on locally-stored consent provided by a third party JS library. In this use case, we can assume that the library vendor and site owner have taken the time explicitly preload resources asynchronously where appropriate, as they must knowingly disable speculative requests.

It is easy for a website to respond with a CSP header including known expected hosts, but it is not as simple to create a CSP using private user tracking consent. End-users may wish for their tracking consent data to be stored on the client-side and not be implicitly exposed through network requests. It is possible to create a client-side JavaScript library (e.g. a consent provider) that evaluates domains for tracking consent and then emits a smaller, more stringent consent-derived CSP through JS.

Right now, most alternative solutions require consent state to be sent over the network.

## API

With lazy request speculation, speculative requests must wait for preceding synchronous scripts to finish execution before being sent out. Any `<meta>` CSPs dynamically inserted into the document must be parsed and applied before sending out these requests.

### `Request-Speculation` HTTP header

Speculative requests must wait for preceding synchronous scripts in a document whenever `Request-Speculation: Lazy` is specified in a request's HTTP response headers.

### `request-speculation` attribute on document element

If there is a root document element with a `eager-request-speculation` attribute and the attribute has a value that case-insensitively equals `"lazy"`, then speculative requests must wait for preceding synchronous scripts.

### Read-only `Document.prototype.requestSpeculation` getter

`document.requestSpeculation` reflects the document's current request speculation setting as either `'eager'` or `'lazy'`. This is a read-only getter.

## Example usage

```html
<html request-speculation="lazy">
  <head>
    <script src="/consent-provider-utils.js"></script>
    <script>
    // Create meta CSP
    const meta = document.createElement('meta');
    meta.httpEquiv = 'Content-Security-Policy';

    // Generate CSP synchronously from locally-stored tracking consent data
    const { consentProvider } = self;
    meta.content = consentProvider
      ? consentProvider.generateCSPFromConsent(localStorage.trackingConsent)
      : 'default-src […];'; // default CSP

    // Enforce CSP on document
    document.head.appendChild(meta).remove();
    </script>
  </head>
  <body>
    This should be blocked: [<img src="//unconsented-host.example"/>]
  </body>
</html>
   
```

## Considered alternatives

### ServiceWorker CSP injection

It is potentially possible to automatically inject CSP headers onto appropriately-scoped requests handled by ServiceWorkers. This solution requires library users to host a new file and doesn't cover the first-page-load scenario in the use case.

## Stakeholder Feedback / Opposition

- Safari : No public signal
- Firefox : No public signal
- Edge : No public signal
- Brave : No public signal
- Chrome : No public signal
