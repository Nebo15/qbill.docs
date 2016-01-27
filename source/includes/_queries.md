# SQL Queries

Internally we use MongoDB, but we that almost every developer knows SQL, and it can be more convenient to get analytical data trough SQL query.

For this purpose we replicate all data into Postgree DB, and give you access to perform all SQL read queries.

<aside class="notice">
Data returned by an SQL query can have a time lag versus direct API calls. Also we don't guarantee any performance for this query, since some API customers can run very complicated JOINS.
</aside>

(TODO: How to share perfomance for each user?)
(TODO: How to limit query complicity?)
