---
layout: default
title: Documentation - MLServer Polymer
---


## [Documentation](#documentation) {#documentation}

[MLServer Operationalization](https://docs.microsoft.com/en-us/r-server/what-is-operationalization) for [Polymer](https://www.polymer-project.org)
Bind operationalization state to properties and dispatch actions from within Polymer Elements.

Polymer is a modern library for creating [Web Components](https://github.com/w3c/webcomponents) within an application. MLServer Operationalization refers to the process of publishing R and Python models to Microsoft MLServer in the form of web services and the consumption of these services within client applications to affect business results. Joining the two concepts together allows developers to create powerful and complex analytics frontend applications faster and simpler. This approach allows the components you build with Polymer to be more focused on functionality than the applications state.

<small><i class="fa fa-info-circle" aria-hidden="true"></i> ***Info***: This documentation presumes an application has been initialised with [`polymer-cli@next`](https://github.com/Polymer/polymer-cli).</small>  
<small><i class="fa fa-info-circle" aria-hidden="true"></i> ***Info***: This documentation won't go into specifics of how to use Polymer, for more information on Polymer read their [documentation](https://www.polymer-project.org/).</small>

---

## [Install](#install) {#install}

```
bower install --save polymer-mls
```

---

## [Usage](#usage) {#usage}

### [Incude](#include) {#include}

```html
<link rel="import" href="../../bower_components/polymer-mls/polymer-mls.html">
```

### [mls-app](#mls-app) {#mls-app}
The `<mls-app>` element initializes and configures your connection to MLServer. The app is permanently initialized once attached and should not be dynamically bound.

__Example usage:__

```html
<mls-app host="http://localhost:12800" cors></mls-app>
```
__Properties:__

- `host`: Defines the MLServer endpoint.
- `cors`: Use cross origin resource sharing (cors). Off by default.

### [mls-auth](#mls-auth) {#mls-auth}
The `<mls-auth>` element is a wrapper around the MLServer authentication API. It notifies successful authentication, provides user information, and handles different types of authentication including _LDAP_ username / password, and _Azure Active Directory_.

__Example usage:__

```html
<!-- Authentication LDAP/AD -->
<mls-auth username="admin" password="secret"></mls-auth>

<!-- Authentication AAD -->
<mls-auth
    authuri="https://login.windows.net"
    tenant="microsoft.com"
    client-id="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
    resource="XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX">
</mls-auth>

<!-- Authentication registration via already authenticated access-token -->
<mls-auth access-token="{{accessToken}}"></mls-auth>

<!-- Force UI model dialog to enter username/password for Authentication via LDAP/AD -->
<mls-auth></mls-auth>
```
__Properties:__

- `username`: (LDAP) The username.
- `password`: (LDAP) The Password.
- `access-token`: Support authentication registration via already authenticated access-token
- `authuri`: (AAD) The authorize endpoint.
- `tenant`: (AAD) Contains an immutable, unique identifier of the directory tenant that issued the token.
- `clientId`: (AAD) The Application Id assigned to your app when you registered it with Azure AD.
- `resource`: (AAD) The App ID URI of the web API secured resource.


### [mls-service](#mls-service) {#mls-service}
 
The `<mls-service>` element is an easy way to interact with a MLServer operationalized service as an object and expose it to the Polymer databinding system.

Any changes to the element's `inputs` properties will trigger a service invocation.

__Example usage:__

```html
<mls-service
   name="transmission"
   inputs="hp,wt"
   outputs="answer">
</mls-service>
```

```R
# -- Assuming a Published service named `transmission` --
manualTransmission <- function(hp, wt) {
    newdata <- data.frame(hp = hp, wt = wt)
    answer <- predict(model, newdata, type = 'response')
    answer
}
```
__Properties:__

- `name`: The operationalized web service name published on the MLServer.
- `version`: (Optional) The operationalized web service version. Default to latest if not provided.
- `inputs`: A csv list of input names as defined by the operationalized web service.
- `outputs`: A csv list of output names as defined by the operationalized web service.
- `params`: (Optional) An object literal that contains service input parameters by name/values. If this property is provided the service will immediately by invoked and the response will be pushed to the callback defined in `on-reponse`. __Note:__ The attribute must be double quoted JSON `params='{ "name": "value" }'`.
- `on-response`: (Optional) callback function. This function will be called when `params` are provided and the service invocation has completed.

### [mls-session](#mls-session) {#mls-session}
 
The `<mls-session>` element is an easy way to create a _R_ or _Python_ session on MLServer and expose it's remote _workspace_ to the Polymer databinding system.

The `<mls-session>` element can work in conjunction with the `<mls-code>` element.

__Example usage:__

```html
<mls-session name="my-r-session" runtime="R"></mls-session>
```

__Properties:__

- `name`: A unique session name for referencing.
- `runtime`: Defines the session's runtime type R or Python.

### [mls-code](#mls-code) {#mls-code}
 
The `<mls-code>` element is an easy way to execute a block of _R_ or _Python_ code on a remote session within the MLServer and expose it's remote _workspace_ to the Polymer databinding system.

__Example usage:__

```html
  <mls-code 
    name="faithful"
    session="my-r-session"
    inputs="n_breaks,individual_obs,density,bw_adjust"
    outputs="plot"
    code="
       hist(faithful$eruptions,
            probability = TRUE,
            breaks = as.numeric(n_breaks),
            xlab = 'Duration (minutes)',
            main = 'Geyser eruption duration');

            if (individual_obs) {
                rug(faithful$eruptions)
            }
                  
            if (density) {
                dens <- density(faithful$eruptions, adjust = bw_adjust);
                lines(dens, col = 'blue');
            }">
        </mls-code>
```

__Properties:__

- `name`: A unique execution name for referencing.
- `session`: The name of the already created _R_ or _Python_ session.
- `code`: The _R_ or _Python_ code block to be executed on the session.
- `inputs`: A csv list of input names as defined by the code block.
- `outputs`: A csv list of output names as defined by the code block.
- `params`: (Optional) An object literal that contains code block input parameters by name/values. If this property is provided the code block will immediately by executed on the session and the response will be pushed to the callback defined in `on-response`. __Note:__ The attribute must be double quoted JSON `params='{ "name": "value" }'`.
- `on-response`: (Optional) callback function. This function will be called when `params` are provided and the code execution has completed.

## [Creating Elements](#creating-elements) {#creating-elements}

Polymer uses _Class Mixins_ to extend and reuse functionality across Elements. Polymer MLServer uses this techinque to connect any extending Element to a MLServer internal data store.

`MLServerMixin` is just a Class Mixin that can be used to mix with `Polymer.Element` or any other Element within an application. A Class Mixin function takes an internal Class definition, extends the given parent Class and returns a new Class.

```html
 <!-- src/demo-app/demo-app.html -->

<link rel="import" href="../../bower_components/polymer/polymer.html">
<link rel="import" href="polymer-mls.html">
<script>
  // Use MLServerMixin to mix Polymer.Element into a new Class
  class CheckBoxDemo extends MLServerMixin(Polymer.Element) {
    static get is() { return 'checkbox-demo'; }
  }
  customElement.define(CheckBoxDemo.is, CheckBoxDemo);
</script>
```

`CheckBoxDemo` now extends both `Polymer.Element` and the underlying Polymer `MLServerMixin` Class.
