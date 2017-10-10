# Nextcloud 12 administration tips/tricks


## File Access Control Mimetype rules

To block specific files you must define the corresponding regex:

```
| Object             | Rule         | Regex                                                           |
| ------------------ | ------------ | --------------------------------------------------------------- |
| Executables        | matches      | /^application\/(octet-stream|x-ms-dos-executable|x-msi)$/i      |
| Compressed files   | matches      | /^application\/(zip|x-zip-compressed)$/i                        |
| Epubs              | is           | application/epub+zip                                            |
```


To identify the exact mime type for your rules:
- upload a sample file (ie: `someRandomEbook.epub`)
- query the NC database to see which mimetype matched
- create a rule accordingly 

Sample query:

```
mysql -u root -p nextcloud_db
Enter password:
[---]

MariaDB [nextcloud_db]> SELECT oc_filecache.name,oc_mimetypes.mimetype FROM oc_filecache,oc_mimetypes where oc_filecache.mimetype=oc_mimetypes.id and oc_filecache.name LIKE '%someRandomEbook%';
+----------------------+----------------------+
| name                 | mimetype             |
+----------------------+----------------------+
| someRandomEbook.epub | application/epub+zip |
| someRandomEbook.doc  | application/msword   |
+----------------------+----------------------+
```

**Important**: the two files are the same, just with a different extension. NC assigns mimetype based on extension, see `resources/config/mimetypemapping.dist.json`


## Trusted domains:

It's possible to enable/restrict access to a specific fqdn, see the `config/config.php` file:

```
'trusted_domains' =>
array (
0 => '172.16.3.33',
1 => 'cloud.mydomain.tld'
),
```



## Cron


Setup a specific [cronjob](https://docs.nextcloud.com/server/12/admin_manual/configuration_server/background_jobs_configuration.html)  under the apache user:

```
crontab -u apache -e
*/15 * * * * php -f /var/www/nextcloud/cron.php
```

Set up Nextcloud accordingly via Admin web interface








