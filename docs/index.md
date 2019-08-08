# Welcome to UCLCSSA API

Welcome to UCLCSSA API's documentation. UCLCSSA API is built using:

- [Node.js](https://nodejs.org/en/) + [Express.js](https://expressjs.com/)
- [UCL API](https://uclapi.com/)
- [WeChat Open Platform](https://open.weixin.qq.com/) + [WeChat MiniProgram](https://developers.weixin.qq.com/miniprogram/dev/framework/)

!!! info "Version"
    The UCLCSSA API documented here is version **1.0.0**.

!!! tip "Source Code"
    Source code for UCLCSSA API available at [UCLCSSA/API-Server](https://github.com/UCLCSSA/API-Server/).

## Endpoint Documentation

A typical endpoint is documentated in the following format:

!!! note "`HTTP_METHOD /endpoint/address?query_parameter={}`"
    Info regarding this endpoint. The `HTTP_METHOD` is one of `GET`, `POST`,
    `PUT` or `DELETE`, e.g. `GET /posts`. For query parameters, they are
    documented in the **Query Parameter(s)** section. If the method is `POST`,
    the requirements of the post body will be documented in **Request Body**
    section. If the endpoint requires authorization, it shall be specified as
    well (the tier of authorization required is either `wechat-registered`,
    `uclapi-registered` or `none`).

    If the only required header is the `Authorization` header, it will be 
    specified in the Authorization section explicitly and will not be placed
    in a separate Request Header(s) section.

    !!! warning "Authorization"
        `wechat-registered`

    **Query Parameter(s)**:

    | Parameter         | Type  | Description            | Constraints     | Default | Required |
    | ----------------- | ----- | ---------------------- | --------------- | ------- | -------- |
    | `query_parameter` | `int` | Some non-negative int. | `> 0 && <= 100` | `1`     | Yes      |

    **Request Body**:

    ```typescript
    {
        keyA: string,
        nestedGroup: {
            xxx: int,
            yyy: int[]
        }
    }
    ```

    | Key               | Type     | Description  | Constraints   | Default | Required |
    | ----------------- | -------- | ------------ | ------------- | ------- | -------- |
    | `keyA`            | `string` | Hello World. | Not empty.    | `''`    | No       |
    | `nestedGroup.xxx` | `int`    | Hello World. | `>= 0`        | `0`     | No       |
    | `nestedGroup.yyy` | `int[]`  | Numbers.     | Length `>= 0` | `[]`    | Yes      |

    ---

    !!! success "Successful response"
        Info on successful response. The HTTP status code, as well as any
        response header and body shall be documented.

        **Status Code**: `200 OK`

        **Response Header(s)**:
        
        ```http
        Content-Type: application/vnd.uclcssa.v1+json
        ```

        **Response Body**:
        
        ```json
        {
            "message": "cheers"
        }
        ```

    !!! failure "Failure response 1"
        Info on failure response. HTTP status code and response header(s) and 
        body shall be documented as well.

        **Status Code**: `403 Forbidden`

        **Response Header(s)**:

        ```http
        Content-Type: application/vnd.uclcssa.v1+json
        ```

        **Response Body**:
        
        ```json
        {
            "message": "Thou shall not pass."
        }
        ```

    !!! failure "Failure response 2"
        Omitted.
