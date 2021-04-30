# Domain Data

All **records** allow for storing arbitrary information in the "data" map:

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

* A **key** that should have a unique meaning. There is a set of reserved keys for typical use, but users are free to create new keys.
* A **value** which must be represented in JSON \([RFC 8259](https://tools.ietf.org/html/rfc8259)\) and encoded in UTF-8.

## Reserved Keys

### Tezos Domains

All keys with the prefix `td:` are reserved for Tezos Domains-related metadata. We currently recognize:

| Key | Meaning | Type | Example |
| :--- | :--- | :--- | :--- |
| **td:ttl** | The time-to-live of the record and an associated reverse record, if any \(in seconds\). If defined, it specifies the maximum time the record should be stored in caches and other secondary-storage mechanisms. | number | `600` |

### OpenID

The prefix `openid:` is reserved for OpenID claims. The values have their respective meanings according to the [OpenID spec](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims). The value types specified in the OpenID spec must be adhered to.

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
        <p><b>openid:&lt;claim&gt;</b>
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

### Gravatar

To provide an avatar representing their account, [Gravatar](https://gravatar.com/) users can equip their Tezos Domain with the MD5 hash of their Gravatar e-mail.

| Key | Meaning | Type | Example |
| :--- | :--- | :--- | :--- |
| **gravatar:hash** | The [MD5 hash of the user's e-mail](https://en.gravatar.com/site/implement/hash/) on Gravatar in hexadecimal format. | string | `"0bc83cb571cd1c50ba6f3e8a78ef1346"` |

### Social media

| Key | Meaning | Type | Example |
| :--- | :--- | :--- | :--- |
| **twitter:handle** | The associated Twitter handle of the domain. | string | `"BillGates"` |
| **instagram:handle** | The associated Instagram handle of the domain. | string | `"nasa"` |

### Developer accounts

| Key | Meaning | Type | Example |
| :--- | :--- | :--- | :--- |
| **github:username** | User's GitHub account name. | string | `"torvalds"` |
| **gitlab:username** | User's GitLab account name. | string | `"foobar"` |
| **keybase:username** | User's Keybase account name. | string | `"foobar"` |

### Source control

| Key | Meaning | Type | Example |
| :--- | :--- | :--- | :--- |
| **project:repository\_url** | Project's Git repository. | string | `"https://gitlab.com/tezos-domains/contracts.git"` |

