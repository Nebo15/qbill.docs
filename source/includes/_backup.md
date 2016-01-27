# Backup

## Download all data

You can download all data of your ```Accounts```, ```Tranfers```, ```Holds``` and ```Fundings``` whenever you want.

Data is available in a ZIP archive with a JSON files that represent your project. There are single endpoint to download all projects. Each of them will be in a separate folder.

```
GET /v1/backup
```

<aside class="notice">
This endpoint have a separated Rate Limits, you can download this data only once a day.
</aside>
