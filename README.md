# Laravel-Dockerized
Laravel Dockerized is a Traefik integrated solution for the deployment of Laravel to production environments. [Full walkthrough can be found here](https://kyle-jeynes.github.io/Laravel-Dockerized/).

This is basically Laravel Forge, free, at your full control.

# Security

I put the application through Nessus to identify the web application CVE's and found the following:

- HSTS Missing From HTTPS Server (RFC 6797)
- SSL Medium Strength Cipher Suites Supported (SWEET32)
- TLS Version 1.0 Protocol Detection

The newest release addresses these security issues.

# Extending

Please visit Traefik documentation to see how to extend the Traefik capabilities and add additional CNAME's etc... It can be achieved using `docker-compose.override.yml` and adding more labels or by creating new directories with its own configuration!

# Features:

- Fail2Ban - See [Fail2Ban](#fail2ban) to get this working
- Traefik Nginx-Ingress
- DNS ACME Propagation To Offload SSL
- Laravel Dockerfile build (from VCS - can easily be added) with
    - NPM (Node) Latest
    - PHP 7.4.16
- Nginx
- MySQL
- Data persistence
- Easy CI/DI Integration

# FAQ

- I get a HTTP over HTTPS error
  Please see [Issue #3](https://github.com/Kyle-Jeynes/Laravel-Dockerized/issues/3) that solves this.
  
- How can I disable CORS across domains?
  You can uncomment the `laravel/nginx.conf` comments and change the domain to your domain.
  
- When will you support other providers?
  Supporting other providers is an adition that will come sooner than later - its just a case of adding the correct `enviroment` arguments to the Traefik container for your specific provider. [Take a look at the docs](https://doc.traefik.io/traefik/v2.0/https/acme/#providers) and you can easily add your own providers, if supported.

# Installation

Laravel Dockerized is an out-the-box solution for Traefik deployments over `docker-compose`.

If you **do not** require `traefik` then skip over step one.

## Step One: Traefik

**NOTE** This container is built to use the [`godaddy`](https://uk.godaddy.com/) provider. If you have an alternative DNS provider, you should read the [Traefik Docs](https://doc.traefik.io/traefik/v2.0/https/acme/#providers) on using your provider. Simply read the linked docs, find the enviroment keys you need to use and ammend the `docker-compose.yml` file within the `traefik` directory to use them `enviroment` keys insteaad.

- Configure your `.env` inside the `traefik` directory to consist of your [provider API keys](https://developer.godaddy.com/keys) and any customisable options (by default all will work). Add your `domain.tld` also.


- Inside the `traefik` directory, run:
  ```bash
  docker-compose up -d --build
  ```

Should you encounter any issues, you can run `docker logs --tail=200 --follow traefik_traefik_1` to debug issues.

## Step Two: Laravel - Traefik will need to be running

**NOTE** You will need to acquire your own Laravel VCS repository. I would suggest using SSH to make automation and CI/CD much easier.

- Clone your repository inside the `laravel` directory and name it `src` - the Dockerfile looks for the `src` directory when building your image.
  ```bash
  # Replace with your website repository
  git clone git@github.com:laravel/laravel.git src
  ```

- Edit your `.env` file for your `laravel/src` project to include the MySQL options that you can set in your `laravel/.env`. You can see an example in `laravel/.example.production.env`.

  **NOTE** Containers auto_discover DNS records - You must specify the "laravel-mysql" as the mysql host in your `laravel/.env`. Do not try use an IPV4 address.


- Build your containers by running the following command:
  ```bash
  docker-compose up -d --build
  ```
  
## Step Three: Installing dependencies & migrating

You can install Composer within your container by executing the composer install command and then run it with `--no-dev`.

**NOTE** The directory is volumed to your local machine so the composer file will not disappear meaning future builds will only require
you to use `composer install --no-dev`. Ensure you quote the command otherwise Ubuntu will assume you're piping it over to PHP rather than passing it as input.

```bash
docker exec -u root laravel_laravel-php_1 "curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && composer install --no-dev"
```

You can migrate/seed your database by executing the `php artisan *` commands within the container too.

```bash
docker exec laravel_laravel-php_1 php artisan migrate
docker exec laravel_laravel-php_1 php artisan db:seed
```

**NOTE** If your `laravel/.env` is in production, you need to pass the `-it` interactive flag or run with `--force`. 

**NOTE** If you're using the default Laravel or your VCS does not copy `.env` because of `.gitignore` you can `vim` whilst inside a container or `cp`. Using the tutorial above, the Laravel repo:

```bash
docker exec -it -u root laravel_laravel-php_1 bash
:/# cp .env.example .env
:/# php artisan key:generate
```

# Fail2Ban

You'll need to remove the symlink that is created from nginx to the `/dev/stdout` by using:

```
cd laravel
docker-compose up -d --build # if you have not done so already
unlink "`docker volume inspect --format '{{ .Mountpoint }}' laravel_nginx-log`/access.log" # unlink the access.log symlink to stdout
docker-compose up -d --force-recreate # recreate with new changes
```

# Credits

Massive shoutout to [@masseyb](https://blog.lazybit.ch/) for his knowledge contributions to make this project available.
