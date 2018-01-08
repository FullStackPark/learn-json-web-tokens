![JWT logo wider](http://i.imgur.com/qDOOu4o.jpg)

# 学习使用 *JSON Web Tokens* (JWT) ,更好的实现*Authentication* !

![dilbert fixed the internet](http://i.imgur.com/cNElVof.jpg)

学习使用 JSON Web Token (JWT) 来使你的 Web 和/或 Mobile 应用更加的*安全* !

[![Build Status][travis-image]][travis-url]
[![Code Climate](https://codeclimate.com/github/dwyl/learn-json-web-tokens/badges/gpa.svg)](https://codeclimate.com/github/dwyl/learn-json-web-tokens)
[![Test Coverage](https://codeclimate.com/github/dwyl/learn-json-web-tokens/badges/coverage.svg)](https://codeclimate.com/github/dwyl/learn-json-web-tokens)
[![Dependency Status](https://david-dm.org/dwyl/learn-json-web-tokens.svg)](https://david-dm.org/dwyl/learn-json-web-tokens)
[![Node.js Version][node-version-image]][node-version-url]

## *为什么*?

JSON Web Tokens (JWTs) 可以使不同的服务(*包括内部/外部的 APP/网站*)可以*更简单地*传递 to _**send read-only signed**_ "_**claims**_".
Claims 可以是*任意* 的你想让别人*读取*的数据,或者*验证*,但是不允许 ***修改***.


> **Note**: *If that sounds buzz-wordy, don't worry, it will all become clear in the next 5 mins of reading!*

## 什么?

> "***JSON Web Token*** *(JWT) is a compact* ***URL-safe*** *means of
> representing claims to be transferred between two parties.
> The claims in a JWT are* ***encoded*** *as a* ***JSON object*** *that is* ***digitally
> signed*** *using JSON Web Signature (JWS)*.  ~ IETF

###  *用人话这么说*

在 Web/Mobile应用中用来标示/验证一个唯一的个体,可以在页面或者API 请求的入口点的**header** 或**url**中增加一个***标准的 Token值***,这个值可以用来标示用户已经登录或者可以访问这个资源.

例如: `https://www.yoursite.com/private-content/?token=eyJ0eXAiOiJKV1Qi.eyJrZXkiOi.eUiabuiKv`

> **注**: 如果这个看起来还是不是很安全,你可以参考下面的[***安全的部分***](https://github.com/FullStackPark/learn-json-web-tokens#q-if-i-put-the-jwt-in-the-url-or-header-is-it-secure).

### 那么 JWT 到底长成个什么*样呢?

-Token 是一串字符串,这一串字符串中包含了一些*加密过的*信息.

-Token 是不同作用的3个部分组成(以点.进行分割).

-下面的字符串为了可以更好的显示已经分成3个不同的行.

```js
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9           // header
.eyJrZXkiOiJ2YWwiLCJpYXQiOjE0MjI2MDU0NDV9      // payload
.eUiabuiKv-8PYk2AkGY4Fb5KMZeorYBLw261JPQD5lM   // signature
```

#### 1. 头

JWT 的第一个部分所采用的加密字符串用来描述一个简单的 JavaScript 对象,描述这个Token 使用了什么样的算法.
1
#### 2. Payload

JWT 的第二部分是 Token 的核心部分. Payload 部分的长度决定于你在 JWT 中包含了多少信息.
一般的经验法则是：将JWT中的最小值存储起来。

#### 3. 签名部分

第三部分, JWT 的最后一个部分是第一个部分和第二部分的内容生成的签名部分,来标示这个 Token 是合法可用的.

### 什么是声明"Claims"?

声明部分包含了 JWT 规范制定的 ***键*** 和 ***对应的值*** :

+ **iss**：令牌的发行者,issuer
+ **exp**：到期时间戳（拒绝已到期的令牌）。 注意：如规范中所定义，这必须以秒为单位。expire
+ **iat**：JWT发布的时间。 可以用来确定JWT的年龄,issue at
+ **nbf**：“不在之前”是令牌将变为活动的未来时间。not before
+ **jti**：JWT的唯一标识符。 用于防止重新使用或重播JWT。
+ **sub**：令牌的主题（很少使用）
+ **aud**：令牌的受众（也很少使用）


+ **iss**: issuer of the token
+ **exp**: the expiration timestamp (reject tokens which have expired). Note: as defined in the spec, this must be in seconds.
+ **iat**: The time the JWT was issued. Can be used to determine the age of the JWT
+ **nbf**: "not before" is a future time when the token will become active.
+ **jti**: unique identifier for the JWT. Used to prevent the JWT from being re-used or replayed.
+ **sub**: subject of the token (rarely used)
+ **aud**: audience of the token (also rarely used)

See: http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html#RegisteredClaimName

# Example [![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/dwyl/learn-json-web-tokens/issues)


让我们深入分析一个简单的例子。
(the *full* source is in the **/example** directory)

> TRY it: https://jwt.herokuapp.com/

## 服务端

例子中使用**node.js http** 服务端在 **/example/server.js** 创建了4个入口点:

1. **/home** : 主页 (不是必须的,但是 **login** 表单在这里.)
2. **/auth** : *authenticate* 游客(如果登录错误或者失败都会返回错误)
3. **/private** : 我们的受限制的资源的入口 - ***特指需要登录后才能查看的资源*** (合法的 SessionToken)才能查看
4. **/logout** : Token 失效和登出用户,将之前的 Token 设置为无效

我们有意的将 **server.js** 设置的非常的简单:

+ 可读性更强
+ 更好维护
+ 更好测试 (所有的帮助类/处理类都可以单独测试)

> note: if you can make it _simpler_, please submit an [issue](https://github.com/dwyl/learn-json-web-tokens/issues) to discuss!

## 帮助类

所有的帮助方法都保存在 **/example/lib/helpers.js**

最重要的2个方法的简化列表如下:

```javascript
// generate the JWT
function generateToken(req){
  var token = jwt.sign({
    auth:  'magic',
    agent: req.headers['user-agent'],
    exp:   Math.floor(new Date().getTime()/1000) + 7*24*60*60; // Note: in seconds!
  }, secret);  // secret is defined in the environment variable JWT_SECRET
  return token;
}
```
 在用户验证过程, ***生成***  JWT token并且返回给用户的方法如下,用户在之后的请求过程中也需要将此 Token 包含在 Header 的 **Authorization** 中



```javascript
// validate the token supplied in request header
function validate(req, res) {
  var token = req.headers.authorization;
  try {
    var decoded = jwt.verify(token, secret);
  } catch (e) {
    return authFail(res);
  }
  if(!decoded || decoded.auth !== 'magic') {
    return authFail(res);
  } else {
    return privado(res, token);
  }
}
```

如果用户在请求私有(private)资源的时候, Token 验证通过的话,会返回 **checks the JWT supplied by the client is valid**,否则的话会返回 **authFail** ***error***

**Note**: *Yes*, *both* these methods are ***synchronous***.
But, given that neither of these methods require *any* **I/O** *or* **Network** requests,
its pretty safe to compute them synchronously.

> Tip: If you're looking for a ***Full Featured*** **JWT Auth Hapi.js plugin** (which does the verification/validation *asynchronously*) for your Hapi.js-based app please check out: [https://github.com/**dwyl/hapi-auth-jwt2**](https://github.com/dwyl/hapi-auth-jwt2)

## 测试

You may have noticed the [![Build Status][travis-image]][travis-url] badge at the *start* of this tutorial.
This is a sign the author(s) are not just *cobbling* code together.
The tests for both the server routes and helper functions are in: **/example/test**

1. /example/test/**functional.js** - *exercises* all the **helper methods** we created in /example/lib/**helpers.js**
[![Test Coverage](https://codeclimate.com/github/dwyl/learn-json-web-tokens/badges/coverage.svg)](https://codeclimate.com/github/dwyl/learn-json-web-tokens)
2. /example/test/**integration.js** - simulates the requests a *user* would send to the server and tests the *responses*.

Please *read* through the tests and *tell us* if anything is unclear!
**Note**: We wrote a basic "***mock***" of the http req/res objects see: /example/test/**mock.js**
Confused/curious about Mocking? Read [When to Mock (by "Uncle Bob")](http://blog.8thlight.com/uncle-bob/2014/05/10/WhenToMock.html)

- - -

## Frequently Asked Questions (*FAQ*)

> ***Got a Question? Ask!*** >> https://github.com/dwyl/learn-json-web-tokens/issues


### Q: If I put the JWT in the *URL* or *Header* is it *secure*?

Good question! The *quick* **answer** is: ***No***.
Unless you are using SSL/TLS (http**s** in your url) to encrypt the connection,
sending the Token [***in-the-clear***](http://en.wikipedia.org/wiki/Plaintext)
is *always* going to be insecure (the token can be intercepted and re-used by a bad person...).
A *naive* "*mitigation*" is to add *verifiable* "claims" to the token
such as checking that the request came from the ***same browser*** (user-agent),
**IP address** or more advanced
"[**browser fingerprints**](http://stackoverflow.com/a/3287761/1148249)"
... http://programmers.stackexchange.com/a/122385

The solution is to *either*:
+ use one-time-use (_single use_) tokens (_which expire after the link has been clicked_) ***or***
+ Don't use url-tokens where high degree of security is required.
(e.g: don't send someone a link which allows them to perform a transaction)

**Use-cases** for a JWT token in a url are:
+ account verification - when you email a person a link after they register on your site. `https://yoursite.co/account/verify?token=jwt.goes.here`
+ password re-set - ensures that the person re-setting the password has access to the email belonging to the account.
 `https://yoursite.co/account/reset-password?token=jwt.goes.here`

Both of these are good candidates for single-use tokens (_which expire after they have been clicked_).

### Q: How do we *Invalidate* sessions?

The person using your app has their **device** (phone/tablet/laptop)
***stolen***. How do you invalidate the token they were using?

The idea behind JWT is that the tokens are ***stateless***
they can be **computed** by any node in a cluster and verified
without a (slow) request to a database.

#### Store the Token in a Database?

##### LevelDB

If your app is *small* or you don't want to have to run a Redis server,
you can get most of the benefits of Redis by using LevelDB: http://leveldb.org/

We can ***either*** store the ***valid*** Tokens in the DB **or**
we can store the ***invalid*** tokens.
Both of these require a round-trip to the DB to check if valid/invalid.
So we prefer to store ***all*** tokens and update the
**valid** property of t from true to false

Example record stored in LevelDB
```json
"GUID" : {
  "auth" : "true",
  "created" : "timestamp",
  "uid" : "1234"
}
```
We would lookup this record by its GUID:

```js
var db = require('level');
db.get(GUID, function(err, record){
  // pseudo-code
  if(record.auth){
    // display private content
  } else {
    // show error message
  }
});
```
see: example/lib/helpers.js **validate** method for detail.

##### Redis

Redis is the *scalable* way of storing your tokens.

If you are *totally* new to Redis read:
+ Intro: http://redis.io/topics/introduction
+ Redis in 30 mins:
http://openmymind.net/2011/11/8/Redis-Zero-To-Master-In-30-Minutes-Part-1/
+ What is Redis? http://www.slideshare.net/dvirsky/introduction-to-redis

Redis ***Scales*** (provided you have the RAM):
http://stackoverflow.com/questions/10478794/more-than-4-billion-key-value-pairs-in-redis

> ***Get Started with Redis today***! [https://github.com/dwyl/**learn-redis**](https://github.com/dwyl/learn-redis)

#### Memcache?

***Quick* answer**: *use **Redis***:
http://stackoverflow.com/questions/10558465/memcache-vs-redis


### Q: Returning Visitor (*no State Preservation between sessions*)

Cookies are stored on the client and are sent by the browser to the server on every request. If the person *closes* their browser, cookies are preserved, so they can continue where they left off without having to log-in again. However, cookies will be sent on **all** requests that match the path and issuing domain, including those for images and css, where it isn't needed.

[`localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window.localStorage) provides a better mechanism for storing tokens during and between browser sessions.

#### Browser-based Applications

There are two options for storing your JWTs:
1. Use ***localStorage*** to store your JWTs on the client side (_means you need to remember to send the JWT in your `authorization` header for subsequent http/ajax requests_)
2. Store your JWT in a cookie (_set and forget_)

> We _obviously_ prefer the cookie-less approach.
But if done right, cookies still have their place in modern web apps!
(_see the Auth0 article on "10 things you should know" in the further reading below_)

##### Useful Links

+ Good ***history*** & overview of **Localstorage**:
http://diveintohtml5.info/storage.html
+ MDN **Window.localStorage**:
https://developer.mozilla.org/en-US/docs/Web/API/Window.localStorage
+ Brief description + basic *examples*:
http://www.html5rocks.com/en/features/storage
+ Will it work for *my* visitors?
http://caniuse.com/#search=localstorage
(**Quick answer**: ***Yes***! IE 8 & above, Android 4.0+, IOS 7.1+, Chrome & Firefox )


#### Programatic (API) Access

Other services accessing your API will have to store the token in a
retrieval system (e.g: Redis or SQLite for mobile apps) and send the token back on each request.

### How to generate secret key?

> "*Apologies if this is mentioned elsewhere. The private key used for signing the tokens, is this the same as a private key generated using ssh-keygen?*" ~ Originally asked by [@skota](https://github.com/skota) see: [dwyl/**hapi-auth-jwt2/issues**/48](https://github.com/dwyl/hapi-auth-jwt2/issues/48)


Since JSON Web Tokens (JWT) are not signed using [***asymmetric encryption***](http://en.wikipedia.org/wiki/Public-key_cryptography) you do not *have* to generate your secret key using ***ssh-keygen***. You can just as easily use a ***strong password*** e.g: https://www.grc.com/passwords.htm provided its ***long and random***. The chance of collision (and thus someone being able to decode your encoded JSON) is pretty low. And if you join two of those **Strong Passwords** (*strings*) together, you'll have a 128bit ASCII String. So the chances of collision are less than the [number of *atoms* in the universe](http://en.wikipedia.org/wiki/Observable_universe#Matter_content_.E2.80.94_number_of_atoms).

To quickly and easily create a secret key using Node's crypto library, run this command.

    node -e "console.log(require('crypto').randomBytes(32).toString('hex'));"

In other words, you *can* use an ***RSA key***, but you don't *have to*.

The main thing you need to remember is: don't share the key with people who are not in your core ("*DevOps Team*") or *accidentally* publish it by committing it to GitHub!




## Which Node.js Module?

A search for "**JSON Web Token**" on NPM:
https://www.npmjs.com/search?q=json+web+token yields ***many*** results!

![npm search for json web token](http://i.imgur.com/ZLN3LlW.png)

### Building a Web App with Hapi.js?

In our efforts to simplify using JWTs in Hapi.js apps,
we wrote this module: https://github.com/dwyl/hapi-auth-jwt2


### General Use in *Other* Node.js Projects

We *highly* recommend using the **jsonwebtoken** module
made by our friends [@auth0](https://twitter.com/auth0)
([the identity/authentication experts](https://auth0.com/about)):
- https://github.com/auth0/node-jsonwebtoken
Which in turn uses:
https://github.com/brianloveswords/node-jws
[![NPM][jsonwebtoken-icon] ][jsonwebtoken-url]

Another great option is: https://github.com/joaquimserafim/json-web-token
by our friend [@joaquimserafim](https://github.com/joaquimserafim)

## Essential Reading (_Background_)

- Original **Specification** (Draft):
https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32
- Great overview from Atlassian:
https://developer.atlassian.com/static/connect/docs/latest/concepts/understanding-jwt.html
- Good intro (ruby-specific examples):
http://www.intridea.com/blog/2013/11/7/json-web-token-the-useful-little-standard-you-haven-t-heard-about
+ Friendlier introduction: http://jwt.io/
+ Getting to know JWT:
https://scotch.io/tutorials/the-anatomy-of-a-json-web-token
- Discussion: https://ask.auth0.com/c/jwt
+ ***How to*** do **stateless authentication** (session-less & cookie-less):
http://stackoverflow.com/questions/20588467/how-to-do-stateless-session-less-cookie-less-authentication


## Further Reading (_Recommended_) [![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/dwyl/learn-json-web-tokens/issues)

+ JWT with Passport.js:
http://stackoverflow.com/questions/20228572/passport-local-with-node-jwt-simple
+ JWT Tokens as API Keys:
https://auth0.com/blog/2014/12/02/using-json-web-tokens-as-api-keys/
+ **10 Thinks you should know** about ***Tokens and Cookies***:
https://auth0.com/blog/2014/01/27/ten-things-you-should-know-about-tokens-and-cookies/#xss-xsrf
+ Information Security discussion:
http://security.stackexchange.com/questions/51294/json-web-tokens-jwt-as-user-identification-and-authentication-tokens
+ Using JWT with node.js (express + backbone):
http://www.sitepoint.com/using-json-web-tokens-node-js/
+ Token-based Authentication with Socket.IO
https://auth0.com/blog/2014/01/15/auth-with-socket-io/
+ JWT Auth *discussion* on Hacker News:
https://news.ycombinator.com/item?id=7084435
+ The Spec but nicer:
http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html
+ Extended (Wiki) article on Claims-based authentication:
http://en.wikipedia.org/wiki/Claims-based_identity
+ Securing Requests with JWT:
http://websec.io/2014/08/04/Securing-Requests-with-JWT.html
+ Avoid Database in authenticating user for each request (stateless):
http://security.stackexchange.com/questions/49145/avoid-hitting-db-to-authenticate-a-user-on-every-request-in-stateless-web-app-ar
+ The Twelve-Factor App: http://12factor.net/ + http://12factor.net/processes
+ Auth in Hapi with JWT: https://medium.com/@thedon/auth-in-hapi-with-jwt-780ce4d072c7#.clgj5lknq

# *Thanks* for Learning with Us!

If you found this quick guide useful, please star it on GitHub!
and re-tweet to share it with others: https://twitter.com/olizilla/status/626487231860080640

[![olizilla tweet](http://i.imgur.com/rCvNvvk.jpg)](https://twitter.com/olizilla/status/626487231860080640 "Please Re-Tweet!")


[jsonwebtoken-icon]: https://nodei.co/npm/jsonwebtoken.png?downloads=true
[jsonwebtoken-url]: https://npmjs.org/package/jsonwebtoken
[npm-image]: https://img.shields.io/npm/v/esta.svg?style=flat
[npm-url]: https://npmjs.org/package/esta
[node-version-image]: https://img.shields.io/node/v/esta.svg?style=flat
[node-version-url]: http://nodejs.org/download/
[downloads-image]: https://img.shields.io/npm/dm/esta.svg?style=flat
[downloads-url]: https://npmjs.org/package/esta
[travis-image]: https://img.shields.io/travis/dwyl/learn-json-web-tokens.svg?style=flat
[travis-url]: https://travis-ci.org/dwyl/learn-json-web-tokens
[coveralls-image]: https://img.shields.io/coveralls/dwyl/learn-json-web-tokens.svg?style=flat
[coveralls-url]: https://coveralls.io/r/dwyl/learn-json-web-tokens?branch=master
[dependencies-url]: https://david-dm.org/dwyl/learn-json-web-tokens
[dependencies-image]: https://david-dm.org/dwyl/learn-json-web-tokens.svg
