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

    | Key         | Type     | Description                  | Constraints | Default | Required |
    | ----------- | -------- | ---------------------------- | ----------- | ------- | -------- |
    | `appId`     | `string` | Obtained from WeChat client. | N/A         | N/A     | Yes      |
    | `appSecret` | `string` | Obtained from WeChat client. | N/A         | N/A     | Yes      |
    | `code`      | `string` | Obtained from WeChat client. | N/A         | N/A     | Yes      |

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

        | Key                 | Type     | Description                      |
        | ------------------- | -------- | -------------------------------- |
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

    | Key     | Type     | Description                    | Constraints          | Default | Required |
    | ------- | -------- | ------------------------------ | -------------------- | ------- | -------- |
    | `email` | `string` | Email to send confirmation to. | Valid email address. | N/A     | Yes      |

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

The `email` does not necessarily have to be an UCL email, since the user will
have to login through their UCL credentials anyway. Also note that
`uclcssaSessionKey` here is only valid if the user has previously registered
through the WeChat registration process.

The API will send a registration email to the user, containing a link to the
`GET /authorize/uclapi` endpoint. Each `uclapiRegistrationCode` is issued to
track UCLAPI registration process, with each `uclapiRegistrationCode` being
valid for *30 minutes*.

!!! note "`GET /authorize/uclapi?uclapiRegistrationCode=...`"
    !!! warning "`Authorization`"
        `none`

    **Query Parameters(s)**:

    | Parameter                | Type     | Description                                                          | Required |
    | ------------------------ | -------- | -------------------------------------------------------------------- | -------- |
    | `uclapiRegistrationCode` | `string` | A registration code issued by the API to track registration process. | Yes      |

    !!! danger "Cache control"
        To protect users, any requests made to this endpoint **must not** be
        cached by the client.

        The API will return a set of headers requiring the client to invalidate
        any caches of this `GET` request, which the client **shall** respect:

        ```http
        HTTP/1.1 301 Redirect
        Cache-Control: no-cache, no-store, must-revalidate
        Pragma: no-cache
        Expires: 0
        ```

    ---

    !!! success "Redirect to UCLAPI"
        Redirects the user to UCLAPI's `https://uclapi.com/oauth/authorize`
        endpoint, with query parameters:

        - `client_id` as provided by API
        - `state` being the user's WeChat `openId` for tracking.
        
        **Status Code**: `301 Redirect`

        **Response Header(s)**:

        ```http
        Location: https://uclapi.com/oauth/authorize?client_id={clientId}&state={openId}
        ```

    !!! failure "Missing `uclapiRegistrationCode`"
        **Status Code**: `400 Bad Request`

        **Response Body**:

        ```json
        {
            "error": "@bad-request/missing-required-query-parameters"
        }
        ```

    !!! failure "Invalid `uclapiRegistrationCode`"
        The user's `uclapiRegistrationCode` may be expired or invalid.

        **Status Code**: `403 Forbidden`

        **Response Body**:

        ```json
        {
            "error": "@forbidden/invalid-uclapi-registration-code"
        }
        ```

If the user agrees to authorize UCLCSSA API to retrieve his/her information
on his/her behalf by loggin in through the user's UCL credentials, UCLAPI will
call the provided callback URL at `GET /authorize/uclapi/callback`.

!!! note "`GET /authorize/uclapi/callback?client_id=...&state=...&result=...&code=...`"
    This endpoint only serves to interface with UCLAPI. Clients may not use this
    endpoint. UCLAPI is whitelisted as origin. See [UCLAPI OAuth#meta](https://uclapi.com/docs/#oauth/meta).

    !!! warning "Authorization"
        `none`

    **Query Parameters**:

    | Parameter       | Type     | Description                      | Required |
    | --------------- | -------- | -------------------------------- | -------- |
    | `client_id`     | `string` | UCLCSSA API `client_id`.         | Yes      |
    | `state`         | `string` | User's WeChat `openId`.          | Yes      |
    | `client_secret` | `string` | UCLCSSA API `client_secret`      | Yes\*    |
    | `code`          | `string` | Voucher to obtain `uclapiToken`. | Yes\*    |

    \* If user denies authorization, `client_secret` and `code` will be missing.
    However, since this endpoint interfaces with UCLAPI, it returns 200 OK to
    recognize that the callback endpoint is in fact successfully invoked.

    ---

    !!! success "Callback URL invoked"
        **Status Code**: `200 OK`

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
