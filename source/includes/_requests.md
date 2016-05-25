# Requests

All incoming requests is logged for your convenience. You can use them for debug or to review as a security log.

Request include, but not limited to: HTTP headers, last 4 digits of access token, all request fields, all response fields.

Outgoing requests is listed in [Webhooks]() section.

All requests are stored and returned in popular [HTTP Archive (HAR)](http://www.softwareishard.com/blog/har-12-spec/) format with one difference, we use snake_case instead of CamelCase as field keys.

## List all Requests
