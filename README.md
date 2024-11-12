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

A Globus Embed is an asset that is rendered to your portal from a configured Globus Connect Server via HTTPS. When rendering these assets, there are [performance](#performance-considerations) and [security](#security-considerations). In both cases, we have made a best effort in embedding assets in a performant and secure way, but be sure to review and understand these considerations before enabling this functionality.

When providing a `FieldDefinition` to a component (e.g. `Result`), the referenced `property` can be processed as an embed by:

- Setting the `property` to a JSON pointer to the location of the asset (the GCS HTTPS URL and path).
- Setting `"type"` to `"globus.embed"`
- Providing the UUID of the `collection` the asset is hosted on via `options`[^1]

[^1]: The `options.collection` configuration is required in order to ensure the proper authorization token is sent along with the HTTPS request for the asset; At this time, the collection cannot be derived from the asset HTTPS URL alone.

https://github.com/globus/example-search-portal-with-embeds/blob/1a4fae30e9e52124e3a8e150b1e18cdc7ab5091a/static.json#L31-L38

<details>
  <summary>
    <code>GMetaResult</code> Reference
  </summary>

```json

{
  "@datatype": "GMetaResult",
  "@version": "2019-08-27",
  "subject": "5352507d-1293-4827-8ba9-c2d4a0eb78d1",
  "entries": [
    {
      "content": {
        "path": "/portal/catalog/dataset_atl",
        "name": "Atlanta International Airport Climate Data (ATL)",
        "id": "5352507d-1293-4827-8ba9-c2d4a0eb78d1",
        "region": "south",
        "sample": "https://g-fe1c1.fd635.8443.data.globus.org/portal/catalog/dataset_atl/1951.csv",
        "tags": [
          "airport"
        ],
        "globus": {
          "transfer": {
            "type": "directory",
            "path": "/portal/catalog/dataset_atl",
            "collection": "a6f165fa-aee2-4fe5-95f3-97429c28bf82"
          }
        }
      },
      "entry_id": null,
      "matched_principal_sets": []
    }
  ]
}
```
</details>


Using this configuration, the portal code will introspect the `Content-Type` returned by the asset and attempt to render the asset for the end user.

In this particular case, because we are referencing a valid CSV, the portal will render the data as a table.

<img width="1061" alt="Screenshot 2024-11-12 at 3 57 05 PM" src="https://github.com/user-attachments/assets/c2bef5f3-a2a0-46fd-9859-caeb1b0b64f1">

For all other `Content-Type` values the portal will attempt to render the asset as an [`<object>` tag](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/object) with the returned `Content-Type` as the [`type` attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/object#type). By using an HTML standard as the default rendering method, the resource will be rendered to the screen based on the end-user's browser environment.

The `type` attribute of the rendered tag can be override by providing an explicit `mime` type in your `FieldDefinition.options`.

<details>
  <summary>Example with <code>options.mime</code></summary>

```json
  {
    "label": "Sample",
    "property": "entries[0].content.sample",
    "type": "globus.embed",
    "options": {
      "collection": "a6f165fa-aee2-4fe5-95f3-97429c28bf82",
      "mime": "application/pdf"
    }
  }
```  
</details>


### Renderers

In addition to the default rendering behaviors of a Globus Embed Field, you can opt-in to specific `renderer` built to process and render referenced assets.

#### Plotly

The Plotly renderer uses the [Plotly JavaScript Open Source Graphing Library](https://plotly.com/javascript/) to render the referenced asset.

To use this functionality `options.renderer` should be set to `plotly` and the sourced asset should be a JSON object that is compatible with [`Plotly.newPlot`](https://plotly.com/javascript/plotlyjs-function-reference/#plotlynewplot) single configuration object parameter.

https://github.com/globus/example-search-portal-with-embeds/blob/1a4fae30e9e52124e3a8e150b1e18cdc7ab5091a/static.json#L39-L47

![Screen Cast 2024-11-12 at 4 09 06 PM](https://github.com/user-attachments/assets/406efc21-7e98-4c05-a358-92e52229ef89)



### Performance Considerations

**Assets that are referenced using the `globus.embed` configuration will be requested using HTTPS to the configured Collection.** Factors such as, the total size of individual embedded assets and the number of embedded assets rendered on a single page will result in HTTPS requests made to in


### Security Considerations

**We have put forth a best effort in supporting embedding assets in your search portal in a secure way, but it is important to note that embedding, untrusted, user-provided content can be dangerous for your end-users.**

Our default protections are based on best practices related to avoiding [Cross Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/) – please be sure to consider these when enabling embedding _and_ making adjustments to the defaults.

#### Default Protections

- A default [Content Security Policy (CSP)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) is set using a `<meta>` tag.
  - You can provide your own value for this tag using `data.attributes.contentSecurityPolicy` in your `static.json`, or setting the value to `false` to remove the `<meta>` tag.
- The default `<object>` based embed is rendered as a child of a [`<iframe>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe) where the [`sandbox`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#sandbox), [`allow`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe#allow), and additional attributes are used to create as much as an isolated runtime environment as possible.


## Optimizaitons

### Requesting HTTPS Scope on Sign In

Globus Connect Server requires tokens to include [specific scopes](https://docs.globus.org/globus-connect-server/v5.4/https-access-collections/#access_tokens_for_https) when requesting assets over HTTPS. By default, when you use the `globus.embed` type the underlying component will attempt to parse authorization errors from the request and prompt the user to address them (where possible). This default implementation often means the first render of an asset will be a consent error the user needs to address (pictured below).

<img width="1030" alt="Screenshot 2024-11-12 at 4 19 03 PM" src="https://github.com/user-attachments/assets/0aa41b7d-cbd4-4451-ae23-b4179449a2a8">

One way to avoid this initial error is to prompt for the GCS-required scopes at _Sign In_ if the are known – the below example adds a known scope to the configured client.

```jsonc
{
  "data": {
    "globus": {
      "application": {
        "client_id": "7442cbd9-2766-42b9-9512-9195b12ed167",
        "scopes": [
          // The UUID here would reference the Collection where your assets are served from.
          "https://auth.globus.org/scopes/a6f165fa-aee2-4fe5-95f3-97429c28bf82/https" 
        ]
      }
    }
  }
}
```

**You very likely want to include this optimization when using `globus.embed`**[^2]. The only time you might not want to use this configuration is if your embedded assets are being loaded from many (dynamic) collections. In these cases, you might not be able to enumerate the scopes as a configuration option, or degrade user experience by asking for unnecessary consent at Sign In.


[^2]: **This** portal does not use this configuration as a way to demonstrate the default behaviors.












