# SQL Queries

Internally we use MongoDB, but we known that almost every developer knows SQL, and it can be more convenient way for you to get analytical data trough SQL query.

For this purpose we replicate all data into [PostgreSQL](www.postgresql.org/) DB, and give you access to perform any kind of SQL read queries.

<aside class="notice">
Data returned by an SQL query can have a time lag versus direct API calls. Also we don't guarantee any performance for this query, since some API consumers can run very complicated JOINS.
</aside>

### Limits

(TODO: How to share perfomance for each user?)
(TODO: How to limit query complicity?)

(Idea) Each second our DB is handling your request will exceed your rate limit by 50 requests. (But we need to guarantee performance in this case.)
