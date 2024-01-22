# Notes for using Odoo17 on an Unraid server via the CA installation

- Installing the Odoo container in Unraid... I didn't need to adjust anything from the defaults in the template
- Installing PostgreSQL was also straight-forward and you'll only need to note down the settings you select here to transpose into the 'odoo.conf' file (detailed below)
- The support thread for this container is located in the Unraid Forums [here](https://forums.unraid.net/topic/150914-support-eurotimmy-odoo17/), please post any questions or contributions


### ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **IMPORTANT NOTE** ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png)

After installing Odoo, return to Unraid to view the container template in 'ADVANCED VIEW'\
We will need to make a change to the `Post Arguments` section to avoid having a security exposure open via a default Admin account\
The `Post Arguments` need to be **removed after the initial first successful boot**
- I have used the `-i base` command as Odoo17 wouldn't initialise it's DB during it's first startup
- The `-i base` command **must be removed** to make for a safe environment

### ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **INSTALL WITHOUT DEMO DATA** ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png)
- To initialise the Odoo17 database without DEMO data being installed by default change your `Post Arguments` to be\
 `-i base --without-demo=all --stop-after-init`
- This will initialise Odoo17, then **NOT install the demo data**, and finally will stop Odoo17 from running.
- Please restart your container **after** removing `-i base --without-demo=all --stop-after-init` from your `Post Arguments`

### ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **INSTALLATION STEPS** ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png)


1. The odoo.conf file needs to be copied in to the `Config Location` path on Unraid server\
The `Config Location` path is selected by you in the Odoo17 settings when installing in Unraid CA\
Your typical `Config Location` path may be at `/mnt/user/appdata/odoo/config`

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

4. Once your container starts, open the Odoo Web UI from your Odoo icon
- &nbsp;&nbsp;&nbsp;Username: admin
- &nbsp;&nbsp;&nbsp;Password: admin


### ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **NPM (Nginx Proxy Manager) - Reverse Proxy settings** ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png)

I am using NPM to reverse proxy my Odoo instance via my own domain name\
In NPM on my `odoo.example.com` domain I use the following in my `location` settings (within the 'Advanced' tab)
``` location / {
    # Put your proxy_pass to your application here
        proxy_pass $forward_scheme://$server:$port;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
```
