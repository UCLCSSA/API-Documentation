# Authentication and Authorization

## Basics

Most API endpoints are ^^protected^^. Users are required to [register](#registration)
in order to gain access to protected API endpoints.

Upon registration, the client side receives a `uclcssaSessionKey` which is used
as the authorization bearer token. Any requests made to protected API endpoints
**must** include the `Authorization` header and the `uclcssaSessionKey` as the
token.

The `uclcssaSessionKey` serves to authenticate the user's identity, and is
associated with WeChat and UCLAPI registration information to be authorized to
use protected endpoints.

## Authorization Header

The `uclcssaSessionKey` shall be passed in as a string, as the value of the
`Authorization` header.

!!! example
    Logging out via [/logout](#logging-out) requires the `Authorization` header.

    ```http
    POST /logout HTTP/1.1
    Accept: application/vnd.uclcssa.v1+json
    Authorization: 936A185CAAA266BB9CBE981E9E05CB78CD732B0B3280EB944412BB6F8F8F07AF
    ```

## Registration

UCLCSSA API utilizes a **two-tiered** registration system.

!!! info "Registration Tiers"
    **TIER 1**: `wechat-registered`

    The user needs to register via [/register/wechat](#wechat-registration) to
    gain `wechat-registered` status. Any WeChat-related functionality requires
    the user to have at minimum this registration tier.

    ---

    **TIER 2**: `uclapi-registered`

    The user needs to register via [/register/uclapi](#uclapi-registration) to
    gain `uclapi-registered` status. Any UCL-related functionality requires the
    user to have this registration tier.

!!! warning
    Each `uclcssaSessionKey` has an associated default validity period of
    **30 days**.

    If the user uses the `uclcssaSessionKey` often within the validity period,
    the key's validity period is automatically *extended*. That is, using the
    key extends its validity period.

    If a `uclcssaSessionKey` is expired, it will be *invalidated* and the user
    **must** re-register with the API. The user's information (posts, profile
    information, etc.) will still be retained unless he/she requests to delete
    his/her associated information explicitly.

### WeChat Registration

### UCLAPI Registration

### Logging Out

A user may choose to terminate his/her session. This can be achieved by
invalidating his/her session. After logging out, the user's associated WeChat
`sessionKey` and UCL API `token` are both nullified.

!!! note "`POST /logout`"
    !!! warning "Authorization"
        `wechat-registered` OR `uclapi-registered`

    !!! success
        **Status Code**: `200 OK`
    
    !!! failure "Missing header"
        Missing `Authorization` header.

        **Status Code**: `400 Bad Request`

        **Response Body**:

        ```json
        {
            "message": "Missing required Authorization header."
        }
        ```
    
    !!! failure "Invalid `uclcssaSessionKey`"
        The provided `uclcssaSessionKey` does not exist in records. It may be
        invalid or expired.

        **Status Code**: `403 Forbidden`

        **Response Body**:

        ```json
        {
            "message": "Invalid uclcssaSessionKey."
        }
        ```

!!! example

    ```http
    POST /logout HTTP/1.1
    Accept: application/vnd.uclcssa.v1+json
    Authorization: 936A185CAAA266BB9CBE981E9E05CB78CD732B0B3280EB944412BB6F8F8F07AF
    ```

    The `Authorization` header and a valid `uclcssaSessionKey` is required so
    the API can find a matching user session.
