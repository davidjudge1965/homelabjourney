# Speedtext tracking my network

I like to know what speed i'm getting from my ISP.  I found a nifty docker container to do that.  The web site also provides instructions for k8s deployment.


## Steps


### Creating an encryption key
A key is required for encryption.  The following command generates a key which can be used in the APP_KEY that will be set later.

```bash
echo -n 'base64:'; openssl rand -base64 32;
```

For ease of retrieval, I have saved that key in a file called APP_KEY.

### Setting up a database
Speed Tracker will need somewhere to store its data.  There are four choices: SQLite, MariaDB, MySQL and Postgres.  For this experiment I will use SQLite.

I used portainer (already deployed on my 'dock' VM) to deploy the container.

I copied into Portainer the configuration that I will need from SpeedtestTracker's  [documentation page for Docker Compose](https://docs.speedtest-tracker.dev/getting-started/installation/using-docker-compose):
```yaml
services:
    speedtest-tracker:
        image: lscr.io/linuxserver/speedtest-tracker:latest
        restart: unless-stopped
        container_name: speedtest-tracker
        ports:
            - 8080:80
            - 8443:443
        environment:
            - PUID=1000
            - PGID=1000
            - APP_KEY=
            - DB_CONNECTION=sqlite
        volumes:
            - /path/to/data:/config
            - /path/to-custom-ssl-keys:/config/keys
```

Then I enhanced the environment variables to suit my needs.  The environment variables are documented [here](https://docs.speedtest-tracker.dev/getting-started/environment-variables).

Beyond those already present in the yaml file, the key variables we need to set are: ADMIN_NAME, ADMIN_EMAIL, ADMIN_PASSWORD.

The admin name, email, password are used to set the initial login user.  I used the following values:
- ADMIN_NAME=David
- ADMIN_EMAIL=david.judge@computer.org
- ADMIN_PASSSWORD=DJAdminPassword

I will update the APP_KEY value with the one I created in the first step.

I will also create the volumes in `~/SpeedTest/data`.

In Portainer, I added the yaml above and started the container.

Now I can get to https://dock:8443 where I have to create a user.

At first I could not see anything - It seems the web page assumes the user is running in dark mode.  I had to select the text to see what was there:
![Alt](./assets/SpeedTracker_Start_page.png)

Selecting the first entry takes me to a sign-in page where I gave the initial user-email/password:

Interestingly, you can set the initial user / email / password as environment variables.  These are documented [here](https://docs.speedtest-tracker.dev/getting-started/environment-variables).  Look for the "ADMIN" entries.

You can also set the speedtest to run on a schedule by setting the SPEEDTEST_SCHEDULE environment variable.  Here's an example to run a speedtest at 12 minutes past every even hour (i.e. 00:12, 02:12, 04:12, etc.).
```
SPEEDTEST_SCHEDULE=12 */2 * * *
```


