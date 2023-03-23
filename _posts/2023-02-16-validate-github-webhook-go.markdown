---
layout: post
title:  "How to validate a Github Webhook"
date:   2023-02-16 20:05:44 +0100
langugage: GO
categories: tutorial
---
To begin with, you need to set up your server and create an endpoint for the webhook. To access the
header, I will utilize the Gin Web Framework. To perform the verification, we require a function that will
extract the request body and the value from the header using the key `X-Hub-Signature-256`. Essentially,
Github uses the body and your key to generate a hash to prevent any brute-forcing of your key. Consequently,
the X-Hub-Signature-256 will always differ. If you want to learn more, you should refer to the [Github documentations][github-docs]

{% highlight go %}
func verifySignature(payload []byte, signature string) bool {
}

func main() {
    r.POST("your/webhook/api", func(c *gin.Context)) {
    payload, _ := io.ReadAll(c.Request.Body)

    if !verifySignature(payload, c.GetHeader("X-Hub-Signature-256")) {
        c.String(http.StatusBadRequest, "signature validation failed")
        return
    }
  }
}
{% endhighlight %}

Don't forget to handle the errors when you are programming yourself.

Initially, we must generate a HMAC hash object and provide our secret key, which we also provided to
Github in the webhook settings. This enables us to construct our payload and authenticate it. The sum function
is subsequently applied to finalize the HMAC computation, producing the MAC as a byte slice. We can then
assemble our string and compare it to the given hash.

{% highlight go %}
const (
      secret = "your secret" //you should load it over an env file or something
  )
              
func verifySignature(payload []byte, signature string) bool {
    mac := hmac.New(sha256.New, []byte(secret))
    mac.Write(payload)
    expectedSignature := "sha256=" + hex.EncodeToString(mac.Sum(nil))
    return hmac.Equal([]byte(signature), []byte(expectedSignature))
}
{% endhighlight %}


[github-docs]: https://docs.github.com/en/webhooks-and-events/webhooks/securing-your-webhooks
