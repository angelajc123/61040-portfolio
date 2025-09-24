# Problem Set 2
## Concept Questions
1. The contexts are for different domains. Strings must be unique within a context, but between context can be the same. The context is the shortURLBase in the URL shortening app.
2. NonceGeneration must store the set of used strings because each generated string must be unique within a context. There must be a one to one mapping of integers to string within a context in the counter case, either you can create a function that takes in a number and returns the corresponding string, or you can use the count itself as the string.
3. One advantage of this is that the shortened URLs are easier to remember. One disadvantage is that there may need to be longer shortened URLS in order to avoid collisions. To modify NonceGeneration, add a set of common dictionary words to the state, and in generate, return a nonce that is not already used by this context that is a combination of the words in the common dictionary.

## Synchronization Questions
1. This is because the generate sync only needs to know the context in order to generate a nonce, not the target URL. In order to register a URL though, you need both the targetURL and the shortURLBase.
2. Sometimes result names are different than their variable names, and listing both disambiguates them. For instance in the first sync, (context: shortURLBase) is written out to show that shortURLBase is bound to context, although shortURLBase was a result and variable name from a different action call.
3. It is because generate and register are invoked on behalf of the user request while setExpiry is invoked on behalf of a UrlShortening action.
4. You do not need to pass shortURLBase into actions anymore. For the first sync, remove shortURLBase from user request, and bind context directly to "bit.ly". For the second synce, remove shortURLBase from the user request and bind "bit.ly" directly to shortURLBase in the invoking action.
5. 
```
sync expireResource
when ExpiringResource.expireResource() : (resource)
then UrlShortening.delete(shortUrl: resource)
```

## Extending the Design
1. 
```
concept TrackAnalytics

purpose record lookup counts for short URLs

principle each lookup increments the count for the short URL

state
  a set of Records with
    shortUrl String
    count Number

actions
  increment (shortUrl: String)
    requires record exists for shortUrl
    effect increases the count for shortUrl by 1

  getCount (shortUrl: String): (count: Number)
    requires record exists for shortUrl
    effect returns the number of lookups for shortUrl
```
```
concept Ownership

purpose associate each short URL with the user who created it

principle after assigning a short Url its owner, only the owner will be able to authorize it

state
  a set of Ownerships with
    shortUrl String
    owner User

actions
  assign (shortUrl: String, owner: User)
    effect associates shortUrl with its creator

  authorize (shortUrl: String, user: User)
    requires shortUrl is owned by user
    effect confirms user owns shortUrl
```
2. 
```
sync assignOwnership
when 
    Request.shortenUrl(user)
    UrlShortening.register (): (shortUrl)
then Ownership.assign (shortUrl, owner: user)
```
```
sync recordLookup
when UrlShortening.lookup (shortUrl)
then Analytics.increment (shortUrl)
```
```
sync viewAnalytics
when
    Request.viewAnalytics (shortUrl, user)
    Ownership.authorize (shortUrl, user)
then Analytics.getCount (shortUrl)
```
3. 
- **Allowing users to choose thier own short URLs:** add new action to UrlShortening (ex. registerCustom(shortUrlSuffix, shortUrlBase, targetUrl)) that required shortUrl to not be taken.
- **Using the “word as nonce”:** add dictionary to state of NonceGenration, add new action generateWord(context) that generates a unique word not already in the context.
- **Including the target URL in analytics:** add targetUrl Number to Records in TrackAnalytics, add action getTargetCounts(targetUrl) : (count) that sums all the lookup counts for shortUrls with targetUrl as their target.
- **Generate short URLs that are not easily guessed:** in the generate action in NonceGeneration, specify generating a highly random string
- **Supporting reporting of analytics to unregistered creators of short URLs:** This feature is not desirable because it would loosen the ownership restriction of getting analytics, possibly leading to a privacy/security risk as anyone could get the analytics of a shortUrl that is not registered with a registered user (not just the original creator of that Url)