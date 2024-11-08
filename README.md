# Example: Serverless Search Portal with Globus Embeds

This repository is an example of the [@globus/template-search-portal](https://github.com/globus/template-search-portal)

You can create your own portal with similar functionality by following the [**Creating Your Own Research Search Portal**](https://github.com/globus/template-search-portal?tab=readme-ov-file#creating-your-own-static-research-search-portal) section in the template repository and then referencing the sections below.

## Background

Metadata in a Globus Search often includes references to assets hosted on a Globus Connect Server. When the Globus Connect Server (and Collection) is configured to support [HTTPS Access](https://docs.globus.org/globus-connect-server/v5.4/https-access-collections/) these assets can be configured to be embedded in search results (e.g. "preview").

## Pre-Requisites

- A [Globus Search Index](https://docs.globus.org/api/search/) you administer.
- A Globus Collection that hosts data relevant to the documents in your Globus Search Index.
- A Search Portal, created following: [**Creating Your Own Research Search Portal**](https://github.com/globus/template-search-portal?tab=readme-ov-file#creating-your-own-static-research-search-portal) in our template repository, **with Authentication enabled.**


## Globus Embeds


`@todo – This section is currently under development – check back soon! @jbottigliero 11/08/2024`


## Security Considerations

**We have put forth a best effort in supporting embedding assets in your search portal in a secure way, but it is important to note that embedding, untrusted, user-provided content can be dangerous for your end-users.**

Our default protections are based on best practices related to avoiding [Cross Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/) – please be sure to consider these when enabling embedding _and_ making adjustments to the defaults.

### Default Protections

- A default [Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) is set using a `<meta>` tag.
  - You can provide your own value for this tag using `data.attributes.contentSecurityPolicy` in your `static.json`, or setting the value to `false` to remove the `<meta>` tag.
- The default `<object>` based embed is rendered as a child of a [`<iframe>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe) where the [`sandbox`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#sandbox), [`allow`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#allow), and additional attributes are used to create as much as an isolated runtime environment as possible.
