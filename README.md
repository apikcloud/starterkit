# Run Odoo locally

## Prerequisites
Modify the docker-compose file to indicate the correct repository and log in to the registry.

    docker login


## Run the stack

    docker compose up

## Restore a backup

> Take care to neutralize the database after restoration. This includes
> scheduled actions, outgoing e-mails and other external providers. A
> standard option is available for this in version 16.0 and higher.

### Standard, database manager
http://localhost:8069/web/database/manager

> There are a number of limitations to take into account, including
> upload size and timeout, especially if a reverse proxy is used.


### With CLI

> Be sure to place the zip in the tree so that the file is accessible in
> the Odoo container, for example the config folder or an extra volume.
> Note that in this case, the mounting point must exist in the image and
> be empty.

Open a shell on the container

    docker compose run odoo bash


#### Odoo Shell

Open the odoo shell :

    odoo shell --no-http

And run the following commands :

    from odoo.service.db import restore_db
    restore_db("<database_name>", "/<path_to_zip_file>", True, [neutralize_database=True])


#### Click-odoo

>  This script allows you to restore databses created by using the Odoo
> web  interface or the backupdb script. This avoids timeout and file
> size  limitation problems when databases are too large.

Run the followings commands :

	pip install click-odoo-contrib
    click-odoo-restoredb -c /etc/odoo/odoo.conf [--neutralize] <database_name> <path_to_file>


