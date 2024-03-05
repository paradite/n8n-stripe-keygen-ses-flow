# n8n-stripe-keygen-ses-flow

n8n flow for Stripe payment link + Keygen + AWS SES for license key issuing that I built for [16x Prompt](https://prompt.16x.engineer/).

Sensitive data has been masked in json file. You need to replace the masked data with your own data.

There is no guarantee that this flow will work for you.

## Overview

![overview](/images/overview.png)

## Prerequisites

- Setup n8n instance on your server or use n8n cloud.

- Setup Stripe Payment Link webhook for the event `checkout.session.completed`.

![stripe webhook](/images/stripe-webhook.png)

- Setup AWS SES and enter credentials for AWS in n8n.

![credentials](/images/credentials.png)

## n8n Flow Steps

### 1. Setup webhook to receive the event from Stripe.

### 2. Use If Node to validate the payment link.

- Check `{{$json["body"]["data"]["object"]["payment_link"]}}` against your own payment link to make sure it's your payment link.

> You might also want to do validation against Stripe API, but I didn't do it for simplicity.

### 3. Generate license key using Keygen using `curl` command.

> Using HTTP Request node should also be possible, but I couldn't make it work.

```sh
curl -X POST https://api.keygen.sh/v1/accounts/123/licenses \
  -H 'Content-Type: application/vnd.api+json' \
  -H 'Accept: application/vnd.api+json' \
  -H 'Authorization: Bearer prod-123' \
  -d '{
        "data": {
          "type": "licenses",
          "relationships": {
            "policy": {
              "data": { "type": "policies", "id": "123" }
            }
          },
          "attributes": {
            "name": "{{$node["Webhook"].json["body"]["data"]["object"]["customer_details"]["email"]}}",
            "metadata": {
              "customerEmail": "{{$node["Webhook"].json["body"]["data"]["object"]["customer_details"]["email"]}}"
            }
          }
        }
      }'
```

### 4. Use a Function node to parse the response from Keygen.

```js
for (item of items) {
  if(item.json.stdout) {
    item.json.stdoutObj = JSON.parse(item.json.stdout);
  }
}

return items;
```

### 5. Send email using AWS SES.

Sample email template:
```
Here is your license key for ABC:

{{ $item("0").$node["Function"].json["stdoutObj"]["data"]["attributes"]["key"] }}

This is an automated email, please do not reply to this email.

For any issues, please contact us at https://abc.com/contact.

Regards,
ABC Team
```

### 6. Done.

## Testing

You can use a pinned node to test the flow.

Alternatively, you can use the Stripe test webhook to test the flow.

## Sample Success Run

![success](/images/success.png)

## License

MIT
