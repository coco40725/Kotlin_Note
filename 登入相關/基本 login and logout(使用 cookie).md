## 使用 JWT 進行 login and logout
login 時將 jwt 產生的 token 存在 cookie中，logout 則將此 cookie 移除。

```kotlin
package com.edu.services

import com.auth0.jwt.JWT
import com.auth0.jwt.algorithms.Algorithm
import com.edu.config.JWTConfig
import com.edu.entities.Account
import com.edu.repositories.AccountRepository
import java.time.LocalDateTime
import java.util.*
import javax.enterprise.context.ApplicationScoped
import javax.inject.Inject
import javax.ws.rs.core.NewCookie


/**
@author Yu-Jing
@create 2023-03-18-下午 10:09
 */
@ApplicationScoped
class AuthService @Inject constructor(
    private val accountRepository: AccountRepository,
    private val jwtConfig: JWTConfig
    ){

    fun login(email: String, name: String, token : String?) : NewCookie? {
        // check the email is vaild or not
        if (false) {
            // do something
            return null
        }

        // first time to log in
        accountRepository.findAccountByEmail(email)?: accountRepository.addAccount(
            Account(
            name = name,
            email = email,
            createTime = LocalDateTime.now(),
            updateTime = LocalDateTime.now())
        )

        // give the token
        token?: return null

        val map: MutableMap<String, Any> = HashMap()
        map["alg"] = jwtConfig.alg()
        map["typ"] = jwtConfig.typ()




        val newToken: String = JWT.create()
            .withHeader(map) // header
            .withClaim("user_email", email) // payload
            .withClaim("user_name", name)
            .withIssuedAt(Date()) // sign time
            .sign(Algorithm.HMAC256(jwtConfig.secret())) // signature

        // save token in the cookie
        return NewCookie(jwtConfig.name(),
            newToken,
            jwtConfig.path(),
            jwtConfig.domain(),
            "create cookie",
            jwtConfig.maxAge(),
            jwtConfig.secured())
    }

    fun logout(): NewCookie{
        return NewCookie("jwt-token","","/","localhost", "delete cookie",0, true)
    }
}

```
