# Rate limiting

## What is it?

Rate limiting prevents us from sending large numbers of requests to a target. It can also be referred to as throttling.

A simple example:

* An application has a login form
* When a request is made to login, the IP is saved and a counter assigned
* If more than 10 attempts are made within 1minute the IP is blocked

### Checklist

* [ ] Can we identify how the rate-limiting is being applied?
* [ ] Can we spoof the a header that's being used
  * [ ] `X-Real-IP`
  * [ ] `X-Forwarded-For`
  * [ ] `X-Originating-IP`
  * [ ] `Client-IP`
  * [ ] `True-Client-IP`
* [ ] Can we use other user agents?
* [ ] Can we use different cookies or session tokens?
* [ ] Can we tamper with HTTP verbs
* [ ] Can we decrease the frequency of requests and leave overnight?
* [ ] Can we create legitimate-looking behaviour
