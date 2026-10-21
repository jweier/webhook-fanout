# Test with Example Success Message

* Open Terminal on Mac
* Replace your Webhook success URL at the top of the code below
* Paste the code into Terminal and hit enter
* You should receive a success message in your webhooks

```
curl -X POST "https://webhook-fanout-success.jayweier.workers.dev/" \
  -H "Content-Type: application/json" \
  --data-binary '{
    "username": "Valor 3.0",
    "avatar_url": "https://imgur.com/RYgVurd.png",
    "embeds": [
      {
        "title": "Checked Out! :tada:",
        "color": 2815384,
        "thumbnail": {
          "url": "https://www.pokemoncenter.com/images/DAMRoot/Full-Size/10051/P12321_72-10979-101_01.jpg"
        },
        "footer": {
          "text": "Valor 3.0 • 10/12/2026 5:13 PM",
          "icon_url": "https://imgur.com/RYgVurd.png"
        },
        "fields": [
          {
            "name": "Site",
            "value": "Pokemon Center US",
            "inline": true
          },
          {
            "name": "Mode",
            "value": "Default",
            "inline": true
          },
          {
            "name": "Product",
            "value": "[(72-10979-101) Snorlax Pokémon Comfy Cuddlers Plush](https://www.pokemoncenter.com/product/72-10979-101)",
            "inline": true
          },
          {
            "name": "Size",
            "value": "N/A",
            "inline": true
          },
          {
            "name": "Color",
            "value": "N/A",
            "inline": true
          },
          {
            "name": "Price",
            "value": "14.99",
            "inline": true
          },
          {
            "name": "Quantity",
            "value": "1",
            "inline": true
          },
          {
            "name": "Profile",
            "value": "|| test bot ||",
            "inline": true
          },
          {
            "name": "Proxy",
            "value": "|| (ISP-Zenu) 113.30.197.0:63421:testuser:testpassword ||",
            "inline": true
          },
          {
            "name": "Order Number",
            "value": "|| P0042148636 ||",
            "inline": true
          },
          {
            "name": "Email",
            "value": "|| test@jayweier.com ||",
            "inline": true
          }
        ]
      }
    ]
  }'
```
