## Success Webhook Code

```
/**
 * ============================================================
 * WEBHOOK DESTINATIONS
 * ============================================================
 *
 * Add/remove webhook URLs here.
 */

const UNMASKED_WEBHOOKS = [];

const MASKED_WEBHOOKS = [
  "https://PUTYOURWEBHOOKURLHERE.com",
  "https://PUTYOURSECONDWEBHOOKURLHERE"
];


/**
 * ============================================================
 * WORKER
 * ============================================================
 */

export default {
  async fetch(request, env, ctx) {
    if (request.method !== "POST") {
      return new Response("Webhook fanout is running", {
        status: 200
      });
    }

    const bodyText = await request.text();

    let payload;

    try {
      payload = JSON.parse(bodyText);
    } catch (error) {
      console.error("Webhook payload was not valid JSON");

      return new Response("Invalid JSON", {
        status: 400
      });
    }


    /**
     * ============================================================
     * CAPTURE ORIGINAL WEBHOOK
     * ============================================================
     *
     * This is the permanent-ish analysis copy stored in
     * Cloudflare Workers Logs.
     *
     * IMPORTANT:
     * This contains the ORIGINAL UNMASKED Valor payload.
     *
     * With Workers Logs persistence enabled and sampling at 100%,
     * every webhook should be available in Observability until it
     * reaches Cloudflare's retention limit.
     */

    const receivedAt = new Date().toISOString();

    const cloudflareRequestId =
      request.headers.get("cf-ray") || null;

    const captureId = crypto.randomUUID();

    console.log({
      log_type: "valor_webhook_capture",
      capture_id: captureId,
      received_at: receivedAt,
      cf_ray: cloudflareRequestId,
      payload: payload
    });


    /**
     * Create a completely separate copy for masking.
     * This ensures the original payload remains untouched.
     */

    const sanitizedPayload = JSON.parse(
      JSON.stringify(payload)
    );


    /**
     * Mask sensitive fields
     */

    if (Array.isArray(sanitizedPayload.embeds)) {
      for (const embed of sanitizedPayload.embeds) {
        if (!Array.isArray(embed.fields)) continue;

        for (const field of embed.fields) {
          if (field.name === "Order Number") {
            field.value = maskOrderNumber(field.value);
          }

          if (field.name === "Email") {
            field.value = maskEmail(field.value);
          }

          if (field.name === "Proxy") {
            field.value = maskProxy(field.value);
          }
        }
      }
    }


    /**
     * Prepare both versions
     */

    const unmaskedBody = JSON.stringify(payload);
    const maskedBody = JSON.stringify(sanitizedPayload);


    /**
     * Log fanout information.
     *
     * Do NOT log unmaskedBody here.
     * The original webhook has already been captured above
     * as a structured valor_webhook_capture event.
     */

    console.log({
      log_type: "webhook_fanout",
      capture_id: captureId,
      received_at: receivedAt,
      unmasked_destinations: UNMASKED_WEBHOOKS.length,
      masked_destinations: MASKED_WEBHOOKS.length
    });


    /**
     * Build unmasked requests
     */

    const unmaskedRequests = UNMASKED_WEBHOOKS.map(
      (url, index) => ({
        name: `UNMASKED_${index + 1}`,
        type: "unmasked",
        request: fetch(url, {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: unmaskedBody
        })
      })
    );


    /**
     * Build masked requests
     */

    const maskedRequests = MASKED_WEBHOOKS.map(
      (url, index) => ({
        name: `MASKED_${index + 1}`,
        type: "masked",
        request: fetch(url, {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: maskedBody
        })
      })
    );


    /**
     * Send everything in parallel
     */

    const allRequests = [
      ...unmaskedRequests,
      ...maskedRequests
    ];

    const results = await Promise.allSettled(
      allRequests.map(item => item.request)
    );


    /**
     * Generate response summary
     */

    const summary = results.map((result, index) => {
      const destination = allRequests[index];

      return {
        destination: destination.name,
        type: destination.type,

        success:
          result.status === "fulfilled" &&
          result.value.ok,

        status:
          result.status === "fulfilled"
            ? result.value.status
            : "request_failed"
      };
    });


    /**
     * Log delivery results.
     *
     * This lets us correlate fanout failures with the original
     * webhook using capture_id.
     */

    console.log({
      log_type: "webhook_fanout_result",
      capture_id: captureId,
      received_at: receivedAt,
      destinations: summary
    });


    return Response.json({
      received: true,
      captureId: captureId,
      unmaskedDestinations: UNMASKED_WEBHOOKS.length,
      maskedDestinations: MASKED_WEBHOOKS.length,
      destinations: summary
    });
  }
};


/**
 * ============================================================
 * MASKING FUNCTIONS
 * ============================================================
 */


/**
 * P0042558636
 * →
 * P00*******36
 */

function maskOrderNumber(value) {
  if (typeof value !== "string") {
    return "|| ***** ||";
  }

  const raw = value
    .replace(/\|\|/g, "")
    .trim();

  if (raw.length <= 5) {
    return "|| ***** ||";
  }

  const masked =
    raw.slice(0, 3) +
    "*".repeat(raw.length - 5) +
    raw.slice(-2);

  return `|| ${masked} ||`;
}


/**
 * test@jayweier.com
 * →
 * t***@j******r.com
 */

function maskEmail(value) {
  if (typeof value !== "string") {
    return "|| *** ||";
  }

  const raw = value
    .replace(/\|\|/g, "")
    .trim();

  const atIndex = raw.indexOf("@");

  if (atIndex <= 0) {
    return "|| *** ||";
  }

  const local = raw.slice(0, atIndex);
  const fullDomain = raw.slice(atIndex + 1);

  const lastDotIndex =
    fullDomain.lastIndexOf(".");

  let domain;
  let tld;

  if (lastDotIndex > 0) {
    domain = fullDomain.slice(
      0,
      lastDotIndex
    );

    tld = fullDomain.slice(
      lastDotIndex
    );
  } else {
    domain = fullDomain;
    tld = "";
  }


  /**
   * test → t***
   */

  const maskedLocal =
    local.charAt(0) +
    "*".repeat(
      Math.max(local.length - 1, 3)
    );


  /**
   * jayweier → j******r
   */

  let maskedDomain;

  if (domain.length <= 2) {
    maskedDomain =
      "*".repeat(domain.length);
  } else {
    maskedDomain =
      domain.charAt(0) +
      "*".repeat(domain.length - 2) +
      domain.charAt(domain.length - 1);
  }


  return (
    `|| ${maskedLocal}@` +
    `${maskedDomain}${tld} ||`
  );
}


/**
 * (ISP-Zenu) 113.30.197.0:63421:username:password
 * →
 * (ISP-Zenu) 113.30.197.0
 */

function maskProxy(value) {
  if (typeof value !== "string") {
    return "|| Proxy Hidden ||";
  }

  const raw = value
    .replace(/\|\|/g, "")
    .trim();

  const match = raw.match(
    /^(\([^)]+\))\s+([^:]+)(?::.*)?$/
  );

  if (!match) {
    return "|| Proxy Hidden ||";
  }

  const label = match[1];
  const host = match[2];

  return `|| ${label} ${host} ||`;
}
```