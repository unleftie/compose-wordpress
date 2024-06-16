# A docker compose for WordPress

## Dependencies

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose plugin](https://docs.docker.com/compose/install/)

## Installation

```bash
git clone https://github.com/unleftie/wordpress-compose.git
cd wordpress-compose
cp .env.example .env
docker compose up -d
```

## Restoring MariaDB data from SQL dump file

Note that you need to fix URL value at **wp_options** table, if it is different from the past one

File **dump_name.sql** should contain old WP database and tables. Value $WORDPRESS_DB_NAME must match this database

```bash
docker exec -i uadopomoga-mariadb sh -c 'exec mariadb -uroot -p"$MARIADB_ROOT_PASSWORD"' < dump_name.sql
docker exec -i uadopomoga-mariadb sh -c 'mariadb -u root -p"$MARIADB_ROOT_PASSWORD" -D $MARIADB_DATABASE -e "GRANT ALL PRIVILEGES ON $MARIADB_DATABASE.* TO $MARIADB_USER;"'
docker exec -i uadopomoga-mariadb sh -c 'mariadb -u root -p"$MARIADB_ROOT_PASSWORD" -D $MARIADB_DATABASE -e "FLUSH PRIVILEGES;"'
docker restart uadopomoga-mariadb
```

## Creating SQL dump file with MariaDB data

```bash
docker exec test-mariadb sh -c 'mariadb-dump --all-databases --debug-info -u root -p"$MARIADB_ROOT_PASSWORD"' > dump_name.sql
docker exec test-mariadb sh -c 'mariadb-dump --all-databases --debug-info -u root -p"$MARIADB_ROOT_PASSWORD" | gzip' > dump_name_$(date +%H-%M_%m-%d-%y).sql.gz
```

## Restoring wp-content data from backup

```bash
docker cp wp-content/ uadopomoga-wordpress:/tmp/wp-content/
docker exec -i uadopomoga-wordpress sh -c 'rm -rf /var/www/html/wp-content'
docker exec -i uadopomoga-wordpress sh -c 'mv /tmp/wp-content/ /var/www/html/wp-content/'
docker exec -i uadopomoga-wordpress sh -c 'chown -R www-data:www-data /var/www/html/wp-content/'
docker restart uadopomoga-wordpress
```

## Creating backup with wp-content data

```bash
docker cp uadopomoga-wordpress:/var/www/html/wp-content wp-content/
```

## WP-CLI usage

```bash
# wp cli version

docker compose run --rm wordpress-cli "cli" "version"
```

## Setup nginx as proxy server

You can find nginx configs for WordPress [here](https://www.digitalocean.com/community/tools/nginx?domains.0.php.wordPressRules=true)

## 📝 License

This project is licensed under the [MIT](LICENSE).
