---
title: JWT and JWK
caption: JWT and JWK authentication
category: servers
keywords: jwt authentication jwt
feature:
  artifact: io.ktor:ktor-auth-jwt:$ktor_version
  method: io.ktor.auth.jwt.jwt
redirect_from:
- /features/authentication/jwt.html
ktor_version_review: 1.0.0
---

Ktor supports [JWT (JSON Web Tokens)](https://jwt.io/), which is a mechanism for authenticating JSON-encoded payloads.
It is useful to create stateless authenticated APIs in the standard way, since there are client libraries for it
in a myriad of languages.

This feature will handle `Authorization: Bearer <JWT>`.

{% include feature.html %}

Ktor has a couple of classes to use the JWT Payload as `Credential` or as `Principal`.

```kotlin
class JWTCredential(val payload: Payload) : Credential
class JWTPrincipal(val payload: Payload) : Principal
```

## Configuring server/routes:

JWT and JWK each have their own method with slightly different parameters. 
Both require the `realm` parameter, which is used in the WWW-Authenticate response header.

## Using a verifier and a validator:

The verifier will use the secret to verify the signature to trust the source.
You can also check the payload within `validate` callback to ensure everything is right and to produce a Principal.

### application.conf:

```kotlin
jwt {
    domain = "https://jwt-provider-domain/"
    audience = "jwt-audience"
    realm = "ktor sample app"
}
```

### JWT auth:

```kotlin
val jwtIssuer = environment.config.property("jwt.domain").getString()
val jwtAudience = environment.config.property("jwt.audience").getString()
val jwtRealm = environment.config.property("jwt.realm").getString()

install(Authentication) {
    jwt {
        realm = jwtRealm
        verifier(makeJwtVerifier(jwtIssuer, jwtIssuer))
        validate { credential ->
            if (credential.payload.audience.contains(jwtAudience)) JWTPrincipal(credential.payload) else null
        }
    }
}

private val algorithm = Algorithm.HMAC256("secret")
private fun makeJwtVerifier(issuer: String, audience: String): JWTVerifier = JWT
        .require(algorithm)
        .withAudience(audience)
        .withIssuer(issuer)
        .build()
```

## Using a JWK provider:

```kotlin
fun AuthenticationPipeline.jwtAuthentication(jwkProvider: JwkProvider, issuer: String, realm: String, validate: (JWTCredential) -> Principal?)
```

```kotlin
val jwkIssuer = "https://jwt-provider-domain/"
val jwkRealm = "ktor jwt auth test"
val jwkProvider = JwkProviderBuilder(jwkIssuer)
            .cached(10, 24, TimeUnit.HOURS)
            .rateLimited(10, 1, TimeUnit.MINUTES)
            .build()
install(Authentication) {
    jwt {
        verifier(jwkProvider, jwkIssuer)
        realm = jwkRealm
        validate { credentials ->
            if (credentials.payload.audience.contains(audience)) JWTPrincipal(credentials.payload) else null
        }
    }
}
```
