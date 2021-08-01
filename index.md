## Laravel-Dockerized Walkthrough

Ensure you have installed Docker and Docker Compose.

Visit [GoDaddy's Developer Portal](https://developer.godaddy.com/) and press API Keys: sign in and select `prod`, then make a note of these keys.

Clone or download the [latest release](https://github.com/Kyle-Jeynes/Laravel-Dockerized/releases/tag/V1.4) from the releases and add your keys to the `traefik/.env` file. Change the `domain.tld` to yours also.

### Building The Traefik Container

Navigate to your `traefik` directory and build your containers.

```bash
cd traefik && docker-compose up -d --build
```

Running `docker-compose ps` should now show you a traefik container running. The health check can be ignored in the future, but on first bootup it should be healthy.

### Using VCS To Grab Your Laravel Project

FTP shouldn't be used in production, so we added the ability to use VCS with ease from Github to host your project files. Clone it into `laravel/src`

```bash
git clone https://github.com/laravel/quickstart-basic.git laravel/src
```

Edit your `laravel/src/.env` file using the `laravel/.production.example.env` as an aid for DDNS to containers then run:

```bash
cd laravel && docker-compose up -d --build
```

### Using Composer And Interacting With The Container


```bash
docker exec -it -u root laravel_laravel-php_1 /bin/bash
curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
composer install --no-dev
php artisan generate:key
php artisan migrate --force
```

### Fixing HTTP over HTTPS issue

Information on the issue [can be found here](https://github.com/Kyle-Jeynes/Laravel-Dockerized/issues/3). You need to alter your `app/Providers/AppServiceProvider.php` and ensure your `laravel/src/.env` is not set to `local`.

```php
public function register() {
    if (env('APP_ENV') !== 'local') {
        $this->app['request']->server->set('HTTPS', true);
    }
}
```

### Fail2Ban For Production

After performing all these tasks, you'll need to remove the symlink for the nginx containers `access.log`. By default, it points to `/dev/stdout` to enable `docker logs` to work. We need to unlink this. Github.io seems to have template issues in code so the below is missing. Please see the actual [index.md](https://github.com/Kyle-Jeynes/Laravel-Dockerized/blob/gh-pages/index.md#fail2ban-for-production) on Github for this code below.

```bash
unlink "`docker volume inspect --format '{{ .Mountpoint }}' laravel_nginx-log`/access.log"
docker-compose up -d --force-recreate # recreate with new changes
```

If you would like the logs to still write to `stdout`, you can run `tail -f` on the `access.log` in the nginx container. Or just use `docker exec -it laravel_fail2ban_1 /bin/bash -c 'tail -f /var/log/access.log'`
