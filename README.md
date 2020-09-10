# Parser Speculation Control explainer

A [Proposal](https://privacycg.github.io/charter.html#proposals)
of the [Privacy Community Group](https://privacycg.github.io/).

## Authors:

- [Eli Grey](https://dangerous.link/virus.exe)

## Participate
- https://github.com/eligrey/parser-speculation-control/issues

## Introduction

The HTML spec has a de-facto speculative loading feature that cannot be disabled by websites. This feature was introduced in all modern browsers to provide a moderate performance boost on the load time of typical webpages at the time. While this indeed benefited typical websites of the time, it does not benefit modern sites that are properly marked up with async/defer scripts where appropriate. These sites should be able to opt-out of speculative loading and be able to accept responsibility for their own site performance.

Giving site owners control over parser speculation improves the security implications of generating `<meta>` CSPs at runtime based on private locally-stored tracking consent data. Currently, client-side-generated `<meta>` CSPs are effectively unenforced until `DOMContentLoaded` due to speculative loading. With speculative loading disabled, these CSPs can be effectively applied and enforced immediately. 

## Motivating Use Cases

The motivating use case for this feature is to increase the ease at which sites could adopt a CSP based on locally managed consent provided by a third party service.

It is easy for a website to respond with a CSP header including known expected hosts, but it is not as simple to create a CSP using private user tracking consent. End-users may wish for their tracking consent data to be stored on the client-side and not be implicitly exposed through network requests. It is possible to create a client-side JavaScript library (e.g. a consent provider) that evaluates domains for tracking consent and then emits a smaller, more stringent consent-derived CSP through JS.

Right now, all alternative solutions require consent state to be sent over the network.

## API

### `Parser-Speculation` HTTP header

Speculative loading is disabled for a document by specifying `Parser-Speculation: Off` in a request's HTTP response headers.

### `no-speculate` element attribute

Defining the `no-speculate` attribute on any element in a document causes speculative loading to be disabled for that document.

## Example usage

```html
<html no-speculate>
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
    This should be blocked: [<img src="//unconsented-host.example">]
  </body>
</html>
   
```

## Considered alternatives

[This should include as many alternatives as you can,
from high level architectural decisions down to alternative naming choices.]

### ServiceWorker CSP injection

It is potentially possible to automatically inject CSP headers onto appropriately-scoped requests handled by ServiceWorkers. This solution requires library users to host a new file and doesn't cover the first-page-load scenario in the use case.

## Stakeholder Feedback / Opposition

- Safari : No public signal
- Firefox : No public signal
- Edge : No public signal
- Brave : No public signal
- Chrome : Positive
