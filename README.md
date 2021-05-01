# Laravel-Dockerized
Laravel Dockerized is a Traefik integrated solution for the deployment of Laravel to production environments.

# Features:

- Traefik Nginx-Ingress
- Traefik SSL Offload
- Traefik DNS ACME Propagation
- Laravel Dockerfile build with
    - NPM (Node) Latest
    - PHP 7.4.16
- Nginx
- MySQL
- Data persistence
- Easy CI/DI Integration

# Installation

Laravel Dockerized is an out-the-box solution for Traefik deployments over `docker-compose`.

If you **do not** require `traefik` then skip over step one.

## Step One: Traefik

**NOTE** This container only supports the [`godaddy`](https://uk.godaddy.com/) provider. If you have an alternative DNS provider, you should read the [Traefik Docs](https://doc.traefik.io/traefik/v2.0/https/acme/#providers) on using your provider.

- Configure your `.env` inside the `traefik` directory to consist of your [provider API keys](https://developer.godaddy.com/keys) and any customisable options (by default all will work).


- Inside the `traefik` directory, run:
  ```bash
  docker-compose up -d
  ```

Should you encounter any issues, you can run `docker logs --tail=200 --follow traefik_traefik_1` to debug issues.

## Step Two: Laravel

**NOTE** You will need to acquire your own Laravel VCS repository. I would suggest using SSH to make automation and CI/CD much easier.

- Clone your repository inside the `laravel` directory and name it `src` - the Dockerfile looks for the `src` directory when building your image.
  ```bash
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
you to use `composer install --no-dev`.

```bash
docker exec -u root laravel_laravel-php_1 curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && composer install --no-dev
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
