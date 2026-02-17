**What was the bug?**

The bug was a lack of a guard against plain objects in the request method of httpClient.ts.

**Why did it happen?**

At first glance, it might seem like TypeScript didn't warn the developer when assigning a plain object to the oauth2Token property, because TypeScript sees the shape of the class and the plain object and matches them. But no — the shape of the class and the object doesn't match here, because the plain object lacks the method asHeader and the getter expired
. So, that leaves us with the real culprit, which is the type of our oauth2Token property: the TokenStatetype — to be more specific, the utility type Record<string, unknown>, which basically just says any object with a string key and whatever value passes. So how does an empty object pass? Well, an empty object doesn't have a key that isn't a string and a value different from unknown, so both our empty object and the plain object with the properties accessToken and expiresAt pass as valid
TokenState types. Where they don't pass is when checked at runtime with the instanceof operator, and so we don't handle them — the request is sent without an Authorization header.

**Why does your fix actually solve it?**

!(this.oauth2Token instanceof OAuth2Token) works because the instanceof operator doesn't compare the shape of objects the same way TypeScript types are compared. It checks the prototype chain at runtime, which will catch that a plain object isn't an instance of
OAuth2Token, triggering the refreshOAuth2
method.

**What's one realistic case / edge case your tests still don't cover?**

What if the access token is an empty string? The token wouldn't be null or expired, so it wouldn't be refreshed, but the Authorization header would be set to "Bearer " — effectively empty.
