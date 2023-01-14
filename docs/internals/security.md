---
sidebar_position: 5
---

# Security

As a micro frontend framework that mainly relies on Spring Security for its security functionality, 
you can deploy multiple Medusa instances internally and have Hydra be the gateway. 

As a gateway it will combine all micro frontends into a single ui experience.
By doing so, you only have to log in once into Hydra and you will be logged into whatever Medusa instance you hit.

Medusa has some preconfigured security features that can be easily set up by using a default base of security configuration.

To set up security rules on the Medusa level using a default base, you can use something like the following code snippet:

``` 
@Bean
public SecurityWebFilterChain securityWebFilterChain(ServerHttpSecurity http) {
    return MedusaSecurity.defaultSecurity(http)
        .authorizeExchange()
        .pathMatchers("/secure/**").authenticated()
        .pathMatchers("/secure/special-role/**").hasRole("SPECIAL")
        .anyExchange().permitAll()
        .and().build();
}
```

This will forward you to a form login if you hit an authenticated-only endpoint.

If Hydra is running and connected, and you access said endpoint via Hydra, the forward to login will be to Hydra's login form. 

This login form can be the default provided one, or you can add a custom one as 'templates/login.html' and it will use that one instead. It uses plain spring security. 

Once logged into the Hydra login form, a JWT token cookie will be set. This token can be interpreted by any Medusa instance you're connected to. It will automatically log you in.
