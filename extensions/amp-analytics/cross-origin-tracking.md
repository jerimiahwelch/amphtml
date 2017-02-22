# Tracking users across origins

AMP pages can be served from either a publisher's website or the AMP Cache.  Because these two websites have different domains, their access to each other’s cookies can be limited in the context of some browsers.  These additional security restrictions mean that some additional effort will be required to maintain the client identification if and when users browse from both contexts.
 
`amp-analytics` makes this effort simpler by providing a [client identifier as a variable substitution](analytics-vars.md#clientid).  The clientId variable can be called with the cid-scope parameter, which is the name of a cookie to read from if the document is loaded from a first-party context.  Otherwise, the clientId variable will be replaced with a cookie set by AMP itself.  At this moment, AMP doesn't provide for direct access to client-side storage (i.e. cookies).

However, by sending the collection request to your own domain you can read and write cookies on those requests, which enables access to both the AMP clientId and your cookies in the same request.  Once the analytics request reaches your server, it can modify the request as needed and respond with a  [302 response](https://en.wikipedia.org/wiki/HTTP_302) code and [set-cookie headers](https://en.wikipedia.org/wiki/HTTP_cookie#Setting_a_cookie), and specify a redirect to another destination.

For example, given an AMP document containing an `amp-analytics` component configured to ping the URL <https://pub.example.com/collect?ampId=${clientId(example-visitor-id)}> then:

* **When the page is viewed on the cache**, the template will be expanded to something like <https://example.com/collect?ampId=a845c9eec2ea>, where `a845c9eec2ea` is an identifier generated by the AMP runtime that is globally unique to the client/document-source-origin tuple (provided visits are separated by less than a year).
* **When the page is viewed on the origin**, what happens depends on whether the `example-visitor-id` cookie is already set.
    * **If `example-visitor-id=7a43391149ed`**, the template will be expanded to <https://example.com/collect?ampId=7a43391149ed>. (That is, `${clientId(example-visitor-id)}` is replaced with the value of the `example-visitor-id` cookie, whatever it is set to.)
    * **If `example-visitor-id` is *not* set**, then the URL will be expanded to something like <https://example.com/collect?ampId=a845c9eec2ea>, where `a845c9eec2ea` is an identifier generated by the AMP runtime that is globally unique to the client/document-source-origin tuple (provided visits are separated by less than a year).
        * Note that the identifier string `a845c9eec2ea` is exactly the same as that generated in the "on cache" state since the document-source-origin is exactly the same in both cases.

Please be informed that combining cookie IDs from two or more different domains might require updating your privacy policy, or obtaining end user consent in some jurisdictions. 