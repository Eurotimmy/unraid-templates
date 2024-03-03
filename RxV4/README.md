# Reactive Resume V4 on an Unraid server via the CA installation
Installing the Reactive Resume V4 container requires the installation of another 4 supporting containers.

I needed to do some configuration to get things working properly, but I am happy I persisted with it because it's a great tool.\
Many thanks to the author of [Reactive Resume](https://github.com/AmruthPillai/Reactive-Resume), please support [Amruth Pillai](https://github.com/AmruthPillai) for the great work! 🥇

**NOTE: This isn't a standalone container when run in Unraid CA**.\
<img src="https://github.com/Eurotimmy/unraid-templates/blob/main/RxV4/screenshots/Unraid%20Stack.png" alt="unraid reactive resume stack" width="200" height="auto">\
\
I refer to the container names shown in the image above by these names throughout the rest of this document:
- `Reactive_Resume_V4`
- `Chrome_RxV4`
- `MinIO_RxV4`
- `PostgreSQL_RxV4`
- `Redis_RxV4`

\
Installing PostgreSQL 15, MinIO, Chrome (browserless) & Redis containers were pretty quick to do, once you know the right settings of course.\
\
The support thread for this container is located in the Unraid Forums [here](https://forums.unraid.net/topic/152057-support-eurotimmy-reactive-resume-v4-rxv4/), please post any questions or ideas / contributions.



### ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **IMPORTANT NOTE** ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png)

**I am currently documenting and screen capturing my setup to put here for others**\
**Feel free to contribute and assist me to do better!!**


### ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png) **Prerequisites** ![#f03c15](https://placehold.co/15x15/f03c15/f03c15.png)

Before we begin working on the containers themselves we need to do some setup in advance.

### (Required) Email > Make a new "App Password" for your Gmail account ([Example](https://www.zdnet.com/article/gmail-app-passwords-what-they-are-how-to-create-one-and-why-to-use-them/) & [Example](https://mariushosting.com/synology-activate-gmail-smtp-for-docker-containers/))
- or similarly for another email provider, you'll need to know your "email username", "email password", "smtp url" & "smtp port"
- This information will be used to create the `SMTP_URL` value in the `Reactive_Resume_V4` container

### (Optional) Reverse Proxy > Add to your domain name records (DNS) if you intend to make this installation public facing
- You will need 2 addresses in your DNS, one for the application you will be accessing to build resumes and one to host the downloadable resume PDFs
- Something along the lines of `cv.example.com` and `cv-store.example.com` may work for this purpose
- `cv.example.com` will be used to create `PUBLIC_URL` value in the `Reactive_Resume_V4` container 
- `cv-store.example.com` will be used to create `STORAGE_URL` value in the `Reactive_Resume_V4` container 

### (Optional) Custom network name
- I opt'ed to make a new network named 'rxv4' by using the command `docker network create rxv4`

### (Recommended) Make some random values we'll use in our containers 
In the Unraid CLI we can make 5x random values, all 15 characters in length by executing this command\
\
```for i in {1..5}; do echo -n "$(openssl rand -base64 6 | head -c 8)$(date +%s | sha256sum | base64 | head -c 7)" | sha256sum | cut -c1-15; done```\
\
(copy the above command, click <img src="https://github.com/Eurotimmy/unraid-templates/blob/main/RxV4/screenshots/Unraid%20CLI.png" alt="unraid open cli icon" width="20" height="auto"> in the top right corner of Unraid, then paste and execute the command)\
\
Record each of the 5 values output by the command above, we'll use these during several of the container configurations
- Value #1 > `CHROME_TOKEN` in `Reactive_Resume_V4` **AND** `TOKEN` in `Chrome_RxV4`
- Value #2 > `ACCESS_TOKEN_SECRET` in `Reactive_Resume_V4` 
- Value #3 > `REFRESH_TOKEN_SECRET` in `Reactive_Resume_V4` 
- Value #4 > `STORAGE_ACCESS_KEY` in `Reactive_Resume_V4` **AND** `MINIO_ROOT_USER` in `MinIO_RxV4`
- Value #5 > `STORAGE_SECRET_KEY` in `Reactive_Resume_V4` **AND** `MINIO_ROOT_PASSWORD` in `MinIO_RxV4`

  
