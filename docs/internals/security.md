---
sidebar_position: 5
---

# Security

As a micro frontend framework that mainly relies on Spring Security for its security functionality, 
you can deploy multiple Medusa instances internally and have Hydra be the gateway. 

As a gateway it will combine all micro frontends into a single ui experience.
By doing so, you only have to log in once into Hydra and you will be logged into whatever Medusa instance you hit.

## Setup
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

## Role mapping
By default, any roles associated with the users in Hydra are also applied to your login session in Medusa. This is a 1-to-1 mapping.

So if you have a Hydra user with the role 'USER', you can use the 'USER' role in Medusa without any further work. The role will be added to the JWT token and applied upon login. If the roles change, you will need a new JWT token.

However, since these are a collection of microservices, you could have roles that don't match between services. For this, there is the concept of role mapping available.

In hydra, you add the following kind of configuration:

```
#Example of mapping a role MANAGER to ADMIN
medusa.hydra.roles.MANAGER.mapped_to=ADMIN
medusa.hydra.roles.MANAGER.service=MyService,MyOtherService
```

This will make it so that a user known in Hydra as the role 'MANAGER', will be known in the services MyService and MyOtherService as 'ADMIN'.

The .service is a comma seperated list, but can also be a regex pattern. For example, you could use '.*' to have a mapping apply to every service.