## 調用 application.properties 的 variable
```kotlin
package com.edu.config

import io.smallrye.config.ConfigMapping

/**
@author Yu-Jing
@create 2023-03-19-下午 01:46
 */
@ConfigMapping(prefix = "jwt")
interface JWTConfig {
    fun alg(): String
    fun typ(): String
    fun name(): String
    fun domain(): String
    fun path(): String
    fun secured(): Boolean
    fun secret(): String
    fun maxAge(): Int
}

```

```properties
//jwt
jwt.alg=HS256
jwt.typ=JWT
jwt.name=jwt-token
jwt.domain=localhost
jwt.path=/
jwt.secured=true
jwt.secret=HELLO123WORLD
jwt.max-age=86400
```