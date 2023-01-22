# Traefik + Let’s Encrypt + Docker Compose

This guide shows you how to deploy your containers behind Traefik reverse-proxy.
It will obtain and refresh HTTPS certificates automatically and it comes with
password-protected Traefik dashboard and a Prometheus / Grafana setup.

## Testing on Your Local Computer

### Step 1: Make Sure You Have Required Dependencies

- Git
- Docker
- Docker Compose

#### Example Installation on Debian-based Systems:

```
sudo apt install git docker.io docker-compose
```

### Step 2: Clone the Repository

```bash
git clone git@github.com:bubelov/traefik-letsencrypt-compose.git
cd traefik-letsencrypt-compose
```

### Step 3: Add Environment Variables

```bash
nano .env
```

```bash
DOMAIN=localhost
EMAIL=admin@localhost
CERT_RESOLVER=
TRAEFIK_USER=admin
TRAEFIK_PASSWORD_HASH=$2y$10$zi5n43jq9S63gBqSJwHTH.nCai2vB0SW/ABPGg2jSGmJBVRo0A.ni
PROMETHEUS_USER=prometheus
PROMETHEUS_PASSWORD_HASH=$2y$10$zi5n43jq9S63gBqSJwHTH.nCai2vB0SW/ABPGg2jSGmJBVRo0A.ni
```

Note that you should leave `CERT_RESOLVER` variable empty if you test your deployment locally. The password is `admin` and you might want to change it before deploying to production.

### Step 4: Set Your Own Password

If you're curious about HTTP basic auth and how it can be used with Traefik, you can read [the full post](https://bubelov.com/blog/basic-auth-reverse-proxy/). Here is the excerpt and it assumes you already installed `htpasswd`:

```bash
htpasswd -nBC 10 admin

New password:
Re-type new password:

admin:$2y$10$zi5n43jq9S63gBqSJwHTH.nCai2vB0SW/ABPGg2jSGmJBVRo0A.ni
```

The output has the following format: `username`:`password_hash`. The username doesn't have to be `admin`, feel free to change it (in the first line).

You can paste the username into the `TRAEFIK_USER` environment variable. The
other part, `hashedPassword`, should be assigned to `TRAEFIK_PASSWORD_HASH`. Now
you have your own `username`:`password` pair. Repeat the previous steps for the
`PROMETHEUS_USER` and `PROMETHEUS_PASSWORD_HASH`.

### Step 5: Add Your Prometheus Configuration
Create a file called `prometheus.yml` and use following configuration and
replace the `<PROMETHEUS_USER>` and `<PROMETHEUS_PLAIN_PASSWORD>` with the credentials
you have created in the `.env` file.

```yml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "traefik"

    basic_auth:
      username: '<PROMETHEUS_USER>'
      password: '<PROMETHEUS_PLAIN_PASSWORD>'
    static_configs:
      - targets: ["host.docker.internal:8082"]
```

### Step 6: Launch Your Deployment

```bash
sudo docker-compose up -d
```

### Step 7: Test Your Deployment

```bash
curl --insecure https://localhost/
```

You can also test it in the browser:

https://localhost/

https://traefik.localhost/

https://grafana.localhost/

## Deploying on a Public Server With Real Domain

Let's say you have a domain `example.com` and it's DNS records point to your production server. Just repeat the local deployment steps, but don't forget to update `DOMAIN`, `EMAIL` and `CERT_RESOLVER` environment variables. In case of `example.com`, your `.env` file should have the following lines:

```bash
DOMAIN=example.com
EMAIL=your@email.com
CERT_RESOLVER=letsencrypt
```

Setting correct email is important because it allows Let’s Encrypt to contact you in case there are any present and future issues with your certificates.

That's it! Let me know if something was unclear to you or if you find any errors.
