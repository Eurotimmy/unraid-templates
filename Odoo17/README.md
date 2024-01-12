**Important notes for using Odoo17 on an Unraid server via the CA installation**

Installing the Odoo container in Unraid... I didn't need to adjust anything from the defaults in the template.
Installing PostgreSQL was also straight-forward and you'll only need to note down the settings you select here to transpose into the 'odoo.conf' file (detailed below).

1. The odoo.conf file needs to be copied in to the `Config Location` path on Unraid server\
The `Config Location` path is selected by you in the Odoo17 settings when installing in Unraid CA\
Your typical `Config Location` path may be at `/mnt/user/appdata/odoo/config`\

2. My [odoo.conf](https://github.com/Eurotimmy/unraid-templates/blob/main/Odoo17/odoo.conf) file if you'd like to use it\
For my path (above), within the Unraid Shell / CLI:
- &nbsp;&nbsp;&nbsp; `cd /mnt/user/appdata/odoo/config`
- &nbsp;&nbsp;&nbsp; `wget https://raw.githubusercontent.com/Eurotimmy/unraid-templates/main/Odoo17/odoo.conf`
- &nbsp;&nbsp;&nbsp; `nano odoo.conf`

3. The `odoo.conf` file needs to be edited **and then** your Odoo Docker container needs to be started / restarted: 
```
  admin_passwd = (This password will become hashed on first start-up, for security reasons)
  db_host = (Use the IP address assigned to the PostgreSQL15 container, in the 'Docker' tab)
  db_port = (Enter what your PostgreSQL15 'Container Port' is set to, from your PostgreSQL install)
  db_user = (Enter what your PostgreSQL15 'POSTGRES_USER' is set to, from your PostgreSQL install)
  db_password = (Enter what your PostgreSQL15 'POSTGRES_PASSWORD' is set to, from your PostgreSQL install)
  dn_name = (Enter what your PostgreSQL15 'POSTGRES_DB' is set to, from your PostgreSQL install)
```

4. Once your container starts, open the Odoo Web UI from your Odoo icon\
- &nbsp;&nbsp;&nbsp;Username: admin
- &nbsp;&nbsp;&nbsp;Password: admin

**Additional notes**

The CA container template can be viewed in 'ADVANCED VIEW' once installed to make a change to the `Post Arguments` section\
The `Post Arguments` should probably be removed **AFTER** the initial first successful boot / login
- I used the `-i base` command as I had a hard time getting Odoo17 to initialise it's DB during it's first startup
- I haven't noticed any issues leaving the `-i base` part in there on subsequent boots, but it runs again on each restart (I removed it to clean up the startup & logs)
