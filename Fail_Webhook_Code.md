## Fail Webhook Code

```
const WEBHOOKS = [
  "https://example.com/webhook-1",
  "https://example.com/webhook-2",
  // Append more webhook URLs here.
];

export default {
  async fetch(request) {
    if (request.method !== "POST") {
      return new Response("Webhook fanout is running", {
        status: 200
      });
    }

    const bodyText = await request.text();

    console.log("Valor webhook payload:", bodyText);

    const body = new TextEncoder().encode(bodyText);
    const destinations = WEBHOOKS.filter(Boolean);

    const requests = destinations.map(url =>
      fetch(url, {
        method: "POST",
        headers: {
          "Content-Type":
            request.headers.get("Content-Type") || "application/json"
        },
        body: body.slice(0)
      })
    );

    const results = await Promise.allSettled(requests);

    const summary = results.map((result, index) => ({
      destination: index + 1,
      success:
        result.status === "fulfilled" &&
        result.value.ok,
      status:
        result.status === "fulfilled"
          ? result.value.status
          : "request_failed"
    }));

    return Response.json({
      received: true,
      destinations: summary
    });
  }
};
```