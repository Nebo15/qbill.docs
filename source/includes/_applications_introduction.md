This API section describes endpoints that is used in our Dashboard. You are free to build any system on top of it. All main interaction rules are the same for consumer and application API.

Main difference is in Authorization and Authentication flows.

## Authorization

To use this API you need to use token with a ```application``` scope. You can receive it in ```/oAuth/basic``` endpoint.

```
TODO
```

<!-- API takes two types of tokens (```application``` and ```project``` tokens), each of them have different access scope. This scope is managed by a object-related Access Control Layer. It means that if you have token that have access to a ```Accounts`` list, than you can manage all records in this list, and all fields in them.

List of objects and token access to them:

Object | ```application``` Token | ```project``` Token
--------- | ----------- | -----------
```Projects``` | Yes | No
```Settings``` | Yes | No
```Backup``` | Yes | No
```Accounts``` | No | Yes
```Fundings``` | No | Yes
```Transfers``` | No | Yes
```Holds``` | No | Yes
```Webhooks``` | No | Yes
```Events``` | No | Yes
```Requests``` | No | Yes
```Currencies``` | No | Yes -->

### Application token

(TODO: Describe root authentication with OTP token. Lets call them ```Managers``` + API for manager login.)

(TODO: Allow creating root user accounts only from our domain.)

# Sign Up

# Sign In
