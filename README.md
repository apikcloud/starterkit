# Starter Kit to run Odoo locally with Docker Compose

## Prerequisites

Docker and Docker Compose must be installed on your system.

Modify the docker-compose file to indicate the correct mount point for your custom addons. In the example, it is set to `./extra-addons`. It means that your custom addons should be placed in the `extra-addons` folder. Only addons present at the root of the mount point will be loaded by the application. If you have already cloned the repository containing the addons, you cat simply set the mount point to that folder (`<custom_path>:/mnt/extra-addons:rw`).


If needed, log in to the registry to pull the images:
```bash
docker login
```

Check and modify the `odoo.conf` file to set your desired configuration.

Fix permissions on the local folders if necessary.

## Run the stack

```bash
docker compose up
```

This will start Odoo and PostgreSQL services.

## Restore a backup

> Take care to disable CRON threads by setting (`max_cron_threads = 0` in odoo.conf) and
> neutralize the database during restoration. This includes
> scheduled actions, outgoing e-mails and other external providers. A
> standard option is available for this in version 16.0 and higher.


The restoration of a database backup can be done in many ways, here is one of them : via the [Database Manager](http://localhost:8069/web/database/manager).

See the documentation for more details, including CLI methods : [Operations > Database Management](https://apikcloud.github.io/docs/#/17-database).





## Open a Shell

```bash
docker compose exec odoo bash
```


## Open the Odoo Shell

From inside the Odoo container, run:

```bash
odoo shell --no-http
```

`-d <database_name>` must be added if database name is not set in odoo.conf (`db_name = <database_name>`).

## Stop the stack

```bash
docker compose down
```

`-v` can be added to remove volumes.


## Additional Services

2 additional services are defined in the `docker-compose.yml` file for convenience:
- **atmoz/sftp** : A simple SFTP server reachable from the Odoo container.
- **maildev** : A mail server that captures all outgoing emails for testing purposes. Accessible at [http://localhost:1080](http://localhost:1080).        

## Additional Resources
- [Odoo Docker Official Documentation](https://hub.docker.com/_/odoo)
- [Odoo Documentation](https://www.odoo.com/documentation)
