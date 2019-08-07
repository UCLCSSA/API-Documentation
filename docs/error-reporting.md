# Error Reporting

To make using UCLCSSA API as pleasent as possible, all API endpoints strive to
have a uniform error reporting format.

Firstly, HTTP status codes are used semantically to convey success or failure.
Any 2XX codes shall indicate successful request. Any 4XX codes shall indicate
failure. Any 5XX codes shall indicate unexpected errors.

For 4XX errors, such as `400 Bad Request`, `403 Forbidden`, any endpoints
returning those status codes shall return a JSON response containing a detailed
`error`.

An `error` is a short string of the format `@http-status/detailed-error-type`.
For example, missing the `Authorization` header in a request made to a protected
endpoint will give an `error` of `@bad-request/missing-authorization-header`.

The client may choose to check `error` for detailed error causes, as
`400 Bad Request` may happen due to different reasons, and `403 Forbidden` may
happen because `uclcssaSessionKey` is either expired or invalid.

!!! info "Typical Error Response"

    ```http
    HTTP/1.1 403 Forbidden
    Content-Type: application/vnd.uclcssa.v1+json
    ```

    ```json
    {
        "error": "@forbidden/uclcssa-session-key-expired"
    }
    ```
