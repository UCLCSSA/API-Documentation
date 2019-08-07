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

**Tier 1** registration. Users who complete the WeChat registration process will be
able to perform basic functionalities (such as forum interactions and posting)
as well as WeChat-specific functionalities.

!!! info "WeChat MiniProgram Login"
    See [MiniProgram Login](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html).

    The WeChat client shall invoke `#!js wx.login()` to obtain a temporary
    `code`, which shall be passed along to UCLCSSA API.

!!! note "`POST /register/wechat`"
    !!! warning "Authorization"
        `none`

    **Request Body**:
    
    ```typescript
    {
        appId: string,
        appSecret: string,
        code: string
    }
    ```

    | Key | Type | Description | Constraints | Default | Required |
    | --------- | ---- | ----------- | ----------- | ------- | -------- |
    | `appId` | `string` | Obtained from WeChat client. | N/A | N/A | Yes |
    | `appSecret` | `string` | Obtained from WeChat client. | N/A | N/A | Yes |
    | `code` | `string` | Obtained from WeChat client. | N/A | N/A | Yes |

    ---

    !!! success
        Successful registration.

        **Status Code**: `200 OK`

        **Response Body**:

        ```json
        {
            "uclcssaSessionKey": "UCLCSSASESSIONKEY"
        }
        ```

        | Key | Type | Description |
        | --- | ---- | ----------- |
        | `uclcssaSessionKey` | `string` | Key representing a user session. |

    !!! failure "Missing required key(s)"
        **Status Code**: `400 Bad Request`

        **Response Body**:

        ```json
        {
            "error": "@bad-request/missing-required-keys"
        }
        ```

### UCLAPI Registration

**Tier 2** registration. Users who upgrade to the `uclapi-registered` tier must
already have the `wechat-registered` tier, that is, already registered through
WeChat.

The user may begin registering for UCLAPI by calling `/register/uclapi`. Note
that a valid `uclcssaSessionKey` is required.

!!! note "`POST /register/uclapi`"
    !!! warning "Authorization"
        `wechat-registered`

    **Request body**:

    ```typescript
    {
        email: string
    }
    ```

    | Key | Type | Description | Constraints | Default | Required |
    | --------- | ---- | ----------- | ----------- | ------- | -------- |
    | `email` | `string` | Email to send confirmation to. | Valid email address. | N/A | Yes |

    ---

    !!! success
        Successful request. A confirmation email will be sent to the specified
        address.

        **Status Code**: `200 OK`

    !!! failure "Missing required email"
        **Status Code**: `400 Bad Request`

        **Response Body**:

        ```json
        {
            "error": "@bad-request/missing-required-keys"
        }
        ```

    !!! failure "Invalid email address"
        **Status Code**: `400 Bad Request`

        **Response Body**:

        ```json
        {
            "error": "@bad-request/invalid-email"
        }
        ```

    !!! failure "Missing `Authorization` header"
        Missing `Authorization` header.

        **Status Code**: `400 Bad Request`

        **Response Body**:

        ```json
        {
            "error": "@bad-request/missing-authorization-header"
        }
        ```

### Logging Out

A user may choose to terminate his/her session. This can be achieved by
invalidating his/her session. After logging out, the user's associated WeChat
`sessionKey` and UCL API `token` are both nullified.

!!! note "`POST /logout`"
    !!! warning "Authorization"
        `wechat-registered` OR `uclapi-registered`

    ---

    !!! success
        **Status Code**: `200 OK`
    
    !!! failure "Missing `Authorization` header"
        Missing `Authorization` header.

        **Status Code**: `400 Bad Request`

        **Response Body**:

        ```json
        {
            "error": "@bad-request/missing-authorization-header"
        }
        ```
    
    !!! failure "Invalid `uclcssaSessionKey`"
        The provided `uclcssaSessionKey` does not exist in records. It may be
        invalid or expired.

        **Status Code**: `403 Forbidden`

        **Response Body**:

        ```json
        {
            "error": "@forbidden/invalid-uclcssa-session-key"
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
