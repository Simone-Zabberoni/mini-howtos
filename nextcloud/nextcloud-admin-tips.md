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


## Trusted domains

It's possible to enable/restrict access to a specific fqdn, see the `config/config.php` file:

```
'trusted_domains' =>
array (
0 => '172.16.3.33',
1 => 'cloud.mydomain.tld'
),
```

## Disable skeleton directory for new users
In  the `config/config.php` file:
```
'skeletondirectory' => '',
```

## Disable "lost password" (ie: when using ldap backend)
In  the `config/config.php` file:
```
 'lost_password_link' => 'disabled',
```

## Log details
In  the `config/config.php` file:
```
  'loglevel' => 1,
```

Set to 0 for maximum details, 2 for default, 4 for disasters only (https://docs.nextcloud.com/server/13/admin_manual/configuration_server/logging_configuration.html)


## Samba configuration

For external CIFS storage it's possible do adapt the smb protocol to the target's supported protocol:
```
cat /etc/samba/smb.conf

[global]
        workgroup = some.group
        security = user

        client use spnego = no
        client max protocol = NT1

......
```


## Cron

Setup a specific [cronjob](https://docs.nextcloud.com/server/12/admin_manual/configuration_server/background_jobs_configuration.html)  under the apache user:

```
crontab -u apache -e
*/15 * * * * php -f /var/www/nextcloud/cron.php
*/15 * * * * php -f /var/www/html/occ  files:scan --all
```

Set up Nextcloud accordingly via Admin web interface








