# Versioning

The current API version is `1.0.0`.

## Versioning Scheme

UCLCSSA API is versioned via [semver 2.0.0](https://semver.org/).

The API version is `{MAJOR}.{MINOR}.{PATCH}`. When changes are made,

- `{MAJOR}` version is incremented when a breaking change is made.
- `{MINOR}` version is incremented when backwards-compatible functionality is
added.
- `{PATCH}` version is incremented when backwards-compatible bug fixes are made.

## Versioning in Requests

Versioning of UCLCSSA API is achieved through *custom media types*. In requests,
only the MAJOR version should be specified, with `v` prepended to the MAJOR
version (i.e. `v1`).

Custom version information can be added to `Accept` header when a request is
made.

!!! info "Media Types"
    All UCLCSSA API media types are formatted as:

    ```text
    application/vnd.uclcssa[.version][+return_format]
    ```

    With `[...]` specifying an *optional* media type information subpart.

    - If `[+return_format]` is omitted, the return format defaults to JSON.

    - If `[.version]` is omitted, the request by default is delegated to the 
    latest API version, which is `v1`.

!!! warning "Specifying API Version in Requests"
    All requests made to UCLCSSA API **should** include specify API version.
    This protects clients from *breaking changes*, as omitting version
    information causes the request to be delegated to the latest API version
    which may contain breaking changes.

!!! example "Typical Request"
    A typical request may look like:

    ```http
    GET /posts HTTP/1.1
    Accept: application/vnd.uclcssa.v1+json
    ```

## Versioning in Responses

All responses returned from UCLCSSA API will include the `Content-Type` header,
which will include the custom media type with version information and payload
format.

!!! example "Typical Responses"

    ```http
    HTTP/1.1 200 OK
    Content-Type: application/vnd.uclcssa.v1+json

    {
        "message": "Successfully logged out."
    }
    ```

    ```http
    HTTP/1.1 400 Bad Request
    Content-Type: application/vnd.uclcssa.v1+json

    {
        "message": "Missing Authorization header."
    }
    ```
