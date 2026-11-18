# Test with Example Password Reset Success Message

* Open Terminal on your Mac.
* Replace the URL in the first line with your success webhook Worker URL.
* Paste the full command into Terminal and press Enter.
* Check your configured webhook destinations for the sample Target password reset message.

```sh
curl -X POST "https://webhook-fanout-success.jay-weier.workers.dev/" \
  -H "Content-Type: application/json" \
  --data-binary '{
  "username": "Valor 3.0",
  "avatar_url": "https://imgur.com/RYgVurd.png",
  "embeds": [
    {
      "title": "Target - Reset Password! :green_circle:",
      "color": 6458623,
      "thumbnail": {},
      "footer": {
        "text": "Valor 3.0 • 11/17/2026 6:33 PM",
        "icon_url": "https://imgur.com/RYgVurd.png"
      },
      "fields": [
        {
          "name": "Credentials",
          "value": "|| test-user@example.com:example-password ||"
        }
      ]
    }
  ]
}'
```
