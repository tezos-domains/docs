# Carthagenet

{% api-method method="post" host="https://carthagenet-api.tezos.domains" path="/graphql" %}
{% api-method-summary %}
GraphQL
{% endapi-method-summary %}

{% api-method-description %}
This endpoint allows you to query Tezos Domains data.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="" type="object" required=true %}
GraphQL query object
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
Cake successfully retrieved.
{% endapi-method-response-example-description %}

```
{
  "data": {
    "domains": {
      "items": [
        {
          "address": "tz1VxMudmADssPp6FPDGRsvJXE41DD6i9g6n",
          "name": "aaa.tez",
          "owner": "tz1VxMudmADssPp6FPDGRsvJXE41DD6i9g6n",
          "level": 2
        },
        {
          "address": null,
          "name": "a.aaa.tez",
          "owner": "tz1VxMudmADssPp6FPDGRsvJXE41DD6i9g6n",
          "level": 3
        },
        {
          "address": null,
          "name": "alice.tez",
          "owner": "tz1Q4vimV3wsfp21o7Annt64X7Hs6MXg9Wix",
          "level": 2
        }
      ]
    }
  },
  "extensions": {}
}
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=429 %}
{% api-method-response-example-description %}
You are hitting rate limits of Tezos Domains endpoint.
{% endapi-method-response-example-description %}

```
Status Code: 429
Retry-After: 58
Content: API calls quota exceeded! maximum admitted 100 per 1m.
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% hint style="info" %}
Example of the GraphQL query object
{% endhint %}

```text
{
  domains {
    items {
      address
      name
      owner
      level
    }
  }
}
```

