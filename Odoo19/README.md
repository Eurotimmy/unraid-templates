# Notes for using Odoo19 on an Unraid server via the CA installation

> [!CAUTION]
> **BETA – NOT YET TESTED ON UNRAID. DO NOT USE WITH PRODUCTION DATA WITHOUT TESTING FIRST.**
>
> This Odoo19 template is new and has not been fully tested on Unraid. It is based on the earlier Odoo templates and the official Odoo 19 Docker image. Test it on a separate, non-production setup first, and keep backups of your PostgreSQL database and filestore.
>
> These templates and notes are provided as-is, free of charge, with no warranty of any kind. You use them at your own risk. I accept no responsibility for any loss of data, downtime or other issues arising from their use, or for any due diligence you choose not to do beforehand.
>
> Please report anything that does or doesn't work in the [support thread](https://forums.unraid.net/topic/150914-support-eurotimmy-odoo17-18-19-20/).

- Installing the Odoo container in Unraid... the template defaults are expected to work as-is (as they did for Odoo18), but this has not yet been confirmed for Odoo19
- Odoo19 requires **PostgreSQL 13 or later** (PostgreSQL 16 or 17 from CA is a good choice)
- Installing PostgreSQL was also straight-forward and you'll only need to note down the settings you select here to transpose into the 'odoo.conf' file (detailed below)
- The support thread for this container is located in the Unraid Forums [here](https://forums.unraid.net/topic/150914-support-eurotimmy-odoo17-18-19-20/), please post any questions or contributions


### ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **IMPORTANT NOTE** ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png)

Permissions on the mapped volumes
I needed to use the `Extra Parameters` in the Unraid template 'ADVANCED VIEW', with the value set as `--user=100:101` due to having some issues with the container being able to write to the 3 mapped volumes

The process I followed was to check the uid and gid in the Odoo19 container with `docker exec -it Odoo19 sh -c 'echo "uid=$(id -u) gid=$(id -g)"'` which returned `uid=100 gid=101`\
I then manually made the folder I had seen in the error logs with `mkdir -p /mnt/user/appdata/odoo19/data/sessions`\
Then to CHOWN the folders with `chown -R 100:101 /mnt/user/appdata/odoo19`\
And, finally CHMOD the folders with `chmod -R 775 /mnt/user/appdata/odoo19`\
I was good from the next restart of the Odoo19 container\
\
After installing Odoo, return to Unraid to view the container template in 'ADVANCED VIEW'\
We will need to make a change to the `Post Arguments` section to avoid having a security exposure open via a default Admin account\
The `Post Arguments` need to be **removed after the initial first successful boot**
- I have used the `-i base` command as Odoo (17 & 18) wouldn't initialise its DB during its first startup without it
- The `-i base` command **must be removed** to make for a safe environment

### ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **DEMO DATA (changed in Odoo19)** ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png)
- Unlike Odoo17 / Odoo18, **Odoo19 does not install demo data by default**, so the default `-i base` gives you a clean database
- If you *want* demo data (handy for a test instance), change your `Post Arguments` to be\
 `-i base --with-demo --stop-after-init`
- This will initialise Odoo19 **with** the demo data, and then stop Odoo19 from running.
- Please restart your container **after** removing `-i base --with-demo --stop-after-init` from your `Post Arguments`
- Alternatively, set `with_demo = True` in `odoo.conf` before the first start-up

### ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **INSTALLATION STEPS** ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png)


1. The odoo.conf file needs to be copied in to the `Config Location` path on Unraid server\
The `Config Location` path is selected by you in the Odoo19 settings when installing in Unraid CA\
Your typical `Config Location` path may be at `/mnt/user/appdata/odoo19/config`

2. My [odoo.conf](https://github.com/Eurotimmy/unraid-templates/blob/main/Odoo19/odoo.conf) file if you'd like to use it\
For my path (above), within the Unraid Shell / CLI:
- &nbsp;&nbsp;&nbsp; `cd /mnt/user/appdata/odoo19/config`
- &nbsp;&nbsp;&nbsp; `wget https://raw.githubusercontent.com/Eurotimmy/unraid-templates/main/Odoo19/odoo.conf`
- &nbsp;&nbsp;&nbsp; `nano odoo.conf`

3. The `odoo.conf` file needs to be edited **and then** your Odoo Docker container needs to be started / restarted: 
```
  admin_passwd = (This password will become hashed on first start-up, for security reasons)
  db_host = (Use the IP address assigned to the PostgreSQL container, in the 'Docker' tab)
  db_port = (Enter what your PostgreSQL 'Container Port' is set to, from your PostgreSQL install)
  db_user = (Enter what your PostgreSQL 'POSTGRES_USER' is set to, from your PostgreSQL install)
  db_password = (Enter what your PostgreSQL 'POSTGRES_PASSWORD' is set to, from your PostgreSQL install)
  db_name = (Enter what your PostgreSQL 'POSTGRES_DB' is set to, from your PostgreSQL install)
```

4. Once your container starts, open the Odoo Web UI from your Odoo icon\
**Change the admin password straight away** (Settings > Users)
- &nbsp;&nbsp;&nbsp;Username: admin
- &nbsp;&nbsp;&nbsp;Password: admin


### ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **NPM (Nginx Proxy Manager) - Reverse Proxy settings** ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png)

I am using NPM to reverse proxy my Odoo instance via my own domain name\
You will need to edit your `odoo.conf` file (in your `Config Location`) to include `proxy_mode = True`\
In NPM on my `odoo.example.com` domain I use the following in my `location` settings (within the 'Advanced' tab)
``` location / {
    proxy_pass $forward_scheme://$server:$port;
    proxy_set_header X-Forwarded-Host $http_host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_redirect off;
}
```
If you run Odoo19 with `workers` above 0 (multi-process mode), the live chat / notification websocket is served on port 8072.
Map the `Live Chat / WebSocket Port` in the template (ADVANCED VIEW) and add a second location in NPM:
```
location /websocket {
    proxy_pass $forward_scheme://$server:8072;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header X-Forwarded-Host $http_host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
}
```
With `workers = 0` (the default) you don't need this.

### ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **Upgrading from Odoo18** ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png)

- Odoo major versions **cannot** be upgraded by just changing the image tag. Pointing an Odoo18 database at Odoo19 will not work
- Install Odoo19 as a **separate** container with its own appdata folders and its own (new) database
- To migrate existing data use the official [Odoo Upgrade service](https://upgrade.odoo.com) or the community [OpenUpgrade](https://github.com/OCA/OpenUpgrade) project, then restore into Odoo19
- Any custom / community addons in `extra-addons` must be Odoo 19 compatible versions
- Back up your PostgreSQL database and filestore before trying any upgrade

### ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **Additional Resources** ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png)

Theres a great GitHub Gist [here](https://gist.github.com/Guidoom/d5db0a76ce669b139271a528a8a2a27f) with well documented options you can use in the `odoo.conf` file if you want greater control
