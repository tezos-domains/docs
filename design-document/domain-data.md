# Domain Data

All **records** and **reverse records** allow for storing arbitrary information in the "data" map:

{% tabs %}
{% tab title="CameLIGO" %}
```ocaml
type data_map = (string, bytes) map
```
{% endtab %}

{% tab title="Michelson" %}
```text
map %data string bytes
```
{% endtab %}
{% endtabs %}

All entries have:

* A **key** which should have a unique meaning. There is a set of reserved keys for typical use, but users are free to create new keys.
* A **value** which must be represented in JSON \([RFC 8259](https://tools.ietf.org/html/rfc8259)\) and encoded in UTF-8.

## Reserved Keys

### Tezos Domains

All keys with the prefix `td:` are reserved for Tezos Domains-related metadata. We currently recognize:

| Key | Meaning | Type | Example |
| :--- | :--- | :--- | :--- |
| **td:ttl** | The time-to-live of the record \(in seconds\). If defined, it specifies the maximum time the record should be stored in caches and other secondary-storage mechanisms. | number | `600` |

### OpenID

The prefix `openid:` is reserved for OpenID claims. The values have their respective meaning according to the [OpenID spec](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims). The value types specified in the spec must be adhered to.

<table>
  <thead>
    <tr>
      <th style="text-align:left">Key</th>
      <th style="text-align:left">Meaning</th>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p><b>openid:*</b>
        </p>
        <p>(e.g. openid:name)</p>
      </td>
      <td style="text-align:left">Any <a href="https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims">OpenID claim</a>
      </td>
      <td style="text-align:left"><em>see OpenID spec</em>
      </td>
      <td style="text-align:left"><code>&quot;Alice Smith&quot;</code>
      </td>
    </tr>
  </tbody>
</table>

