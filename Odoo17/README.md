Important notes for using Odoo17 on an Unraid server via the CA installation

1. The odoo.conf file needs to be copied in to the `Config Location` path on Unraid server
- The `Config Location` path is selected by you in the Odoo17 settings when installing in Unraid CA
- Your typical `Config Location` path may be at `/mnt/user/appdata/odoo/config`
- Here's my example [odoo.conf](https://github.com/Eurotimmy/unraid-templates/blob/main/Odoo17/odoo.conf) file
2. The odoo.conf file should be edited: **(highly recommended!)** 
```
  admin_passwd = (This password will become hashed on first start-up, for security reasons)
  db_host = (Use the IP address assigned to the PostgreSQL15 container, in the 'Docker' tab)
  db_port = (Enter what your PostgreSQL15 'Container Port' is set to, from your PostgreSQL install)
  db_user = (Enter what your PostgreSQL15 'POSTGRES_USER' is set to, from your PostgreSQL install)
  db_password = (Enter what your PostgreSQL15 'POSTGRES_PASSWORD' is set to, from your PostgreSQL install)
  dn_name = (Enter what your PostgreSQL15 'POSTGRES_DB' is set to, from your PostgreSQL install)
```
3. The CA container template can be viewed in 'ADVANCED VIEW' once installed, the `Post Arguments` should probably be removed **AFTER** the initial first successful boot / login
- I used the `-i base` command as I had a hard time getting Odoo17 to initialise it's DB during it's first startup
- I haven't noticed any issues leaving the `-i base` part in there on subsequent boots, but it runs again on each restart (I removed it to clean up the startup & logs)
4. The default database name the Odoo docker container is looking for is `db`
- This is just 'to note' as it took me some time to realise it...😴
