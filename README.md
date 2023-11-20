
# GITLAB COMMUNITY EDITION 11.10.4 TO LATEST VERSION UPGRADE 

Upgrading GitLab to the latest version involves several steps to ensure a smooth transition and to minimize the risk of data loss. Please note that the steps may vary slightly depending on your operating system and deployment method. Here's a general guide to upgrade GitLab from version 11.10.4 to the latest version


## Acknowledgements

 - [UPGRADE PATH OVERVIEW](https://gitlab-com.gitlab.io/support/toolbox/upgrade-path/?current=11.10.8&distro=centos&auto=true&edition=ce&downtime=true) Review each step and the version specific upgrade notes
 - [UPGRADE PATH DOCS](https://docs.gitlab.com/ee/update/#upgrade-paths)
 - [WHAT'S NEW 11.10 > 16.6](https://gitlab-com.gitlab.io/cs-tools/gitlab-cs-tools/what-is-new-since/?tab=features&minVersion=11_10)
 - [CHECK FOR BACKGROUND MIGRATIONS BEFORE UPGRADING](https://docs.gitlab.com/ee/update/index.html#check-for-background-migrations-before-upgrading)

## Prerequisites

- Ensure that you have sudo privileges on the system.
- This guide assumes a clean installation without any previous PostgreSQL data.
- GitLab installed and running
- Administrative privileges (sudo)
- PostgreSQL 15 installed

## PostgreSQL Installation Guide :

  * Install the PostgreSQL YUM repository:

    ```bash
    sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
    ```

  * Install the PostgreSQL 15 server and its additional components:

    ```bash
    sudo yum install -y postgresql15-server
    sudo yum install -y postgresql15-contrib
    ```

  * If the PostgreSQL data folder is not empty, remove its contents:

    ```bash
    sudo rm -rf /var/lib/pgsql/15/data
    ```

  * Initialize the PostgreSQL 15 database:

    ```bash
    sudo /usr/pgsql-15/bin/postgresql-15-setup initdb
    ```

  * Start the PostgreSQL 15 service:

    ```bash
    sudo systemctl start postgresql-15
    ```

  * Restart the PostgreSQL 15 service (optional, but recommended):

    ```bash
    sudo systemctl restart postgresql-15
    ```

  * Enable PostgreSQL 15 to start on system boot:

    ```bash
    sudo systemctl enable postgresql-15
    ```

  * If needed, stop the PostgreSQL service for any reason:

    ```bash
    sudo systemctl stop postgresql-15
    ```
   # GitLab CE Installation Guide

 This guide provides step-by-step instructions for installing GitLab Community Edition (CE) version 11.10.4 on a system using `yum`.

  ## Prerequisites

- **Operating System:** Ensure that you are running a supported operating system.
- **Root Access:** You should have root or sudo privileges to perform the installation.

## Installation Steps :

  * Before installing GitLab CE, make sure your system packages are up to date.
 
    ```bash
    sudo yum update -y
    ```

# GitLab CE Upgrade to Version 11.11.8:
    sudo yum install -y gitlab-ce-11.10.4

  * Once the installation is complete, run the following command to reconfigure GitLab:

    ```bash
    sudo gitlab-ctl reconfigure
    ```

This command will configure and start the necessary services for GitLab.

## Access GitLab :

After the configuration is complete, you can access GitLab by navigating to `http://your-server-ip` in your web browser. The default login credentials are:

- **Username:** `root`
- **Password:** The password you specified during the installation.

  * Additional Configuration

- **Security:** It is recommended to configure SSL for secure communication. Refer to the GitLab documentation for SSL setup instructions.

- **Backup:** Regularly backup your GitLab instance to prevent data loss. Refer to the GitLab documentation for backup and restore instructions.

# Configuring Repository and External PostgreSQL Database in GitLab

If you encounter any issues during the installation, refer to the [GitLab documentation](https://docs.gitlab.com/) for troubleshooting guides and community support.

## Location: nano /etc/gitlab/gitlab.rb

``` 
  # Changes:

  ### For setting up a different data storing directory
  git_data_dirs({
    "default" => {
      "path" => "/home/git_database/git-data"
    }
  })

  ### GitLab database settings
  gitlab_rails['db_adapter'] = "postgresql"
  gitlab_rails['db_encoding'] = "utf8"
  gitlab_rails['db_database'] = "gitlabhq_production"
  gitlab_rails['db_username'] = "postgres"
  gitlab_rails['db_password'] = "gitlab@123"
  gitlab_rails['db_host'] = "localhost"
  gitlab_rails['db_port'] = "5432"
```
## Assign PASSWORD to postgres user for security
```
  sudo su postgres
  psql
  ALTER USER postgres PASSWORD 'gitlab@123';
```
## To integrate an external PostgreSQL database with your GitLab instance, follow these steps:

  * Open the `pg_hba.conf` file for editing using the Nano text editor:
    ```bash
    nano /var/lib/pgsql/15/data/pg_hba.conf
    ```

  * Edit the `pg_hba.conf` file to include the following configurations:

    ```conf
    # TYPE  DATABASE        USER            ADDRESS                 METHOD

    # "local" is for Unix domain socket connections only
    local   all             all                                     md5
    # IPv4 local connections:
    host    all             all             127.0.0.1/32            md5
    # IPv6 local connections:
    host    all             all             ::1/128                 md5
    # Allow replication connections from localhost, by a user with the
    # replication privilege.
    #local   replication     postgres                                peer
    #host    replication     postgres        127.0.0.1/32            ident
    ```

    Ensure to uncomment or modify the lines according to your specific requirements.

  * After making the necessary changes, save the file.

  * Restart the PostgreSQL service to apply the changes:
    ```bash
    sudo systemctl restart postgresql-15
    ```
This configuration allows GitLab to connect to the external PostgreSQL database securely. Make sure to replace `15` with the appropriate version number if your PostgreSQL version is different.

# Troubleshooting PG::ConnectionBad Error: SCRAM Authentication

If you encounter the PG::ConnectionBad error with the message "SCRAM authentication requires libpq version 10 or above," follow these steps to resolve it:

  * Open the `postgresql.conf` file for editing using the Nano text editor:
    ```bash
    nano /var/lib/pgsql/15/data/postgresql.conf
    ```

  * Locate the section for authentication configurations.

  * Update the `password_encryption` setting to use `md5`:
    ```conf
    # - Authentication -
    password_encryption = md5
    ```

   Save the file after making the change.

  * Restart the PostgreSQL service to apply the changes:
    ```bash
    sudo systemctl restart postgresql-15
    ```

  * Switch to the PostgreSQL user:
    ```bash
    sudo su postgres
    ```

  * Access the PostgreSQL interactive terminal:
    ```bash
    psql
    ```

  * Change the password for the `postgres` user:
    ```sql
    ALTER USER postgres PASSWORD 'gitlab@123';
    ```

   Replace `'gitlab@123'` with your desired password.

  * Exit the PostgreSQL terminal:
    ```sql
    \q
    ```

  * Exit the PostgreSQL user session:
    ```bash
    exit
    ```

These steps ensure that the PostgreSQL configuration is updated to use MD5 password encryption and that the `postgres` user's password is set appropriately.

Make sure to adjust version numbers and passwords as needed for your specific environment.

# Import Data from External PostgreSQL

If you have an external PostgreSQL database and want to import data into GitLab, follow these steps:

  * Execute the following command to import the data from the external SQL file (`gitlabhq_production.sql` in this example):
    ```bash
    sudo -u postgres psql -f /home/git_database/gitlabhq_production.sql
    ```

  * After importing the data, reconfigure GitLab:
    ```bash
    sudo gitlab-ctl reconfigure
    ```

## Allow Privileges for `gitlabhq_production` in PostgreSQL

To grant privileges to the PostgreSQL user for the GitLab database:

  * Switch to the PostgreSQL user:
    ```bash
    sudo su postgres
    ```

  * Access the PostgreSQL interactive terminal:
    ```bash
    psql
    ```

  * Grant all privileges on the `gitlabhq_production` database to the `postgres` user:
    ```sql
    GRANT ALL PRIVILEGES ON DATABASE gitlabhq_production TO postgres;
    ```

  * Optionally, if needed, set the login for the `gitlab` role:
    ```sql
    -- Uncomment the following line to allow login for the "gitlab" role
    -- ALTER ROLE "gitlab" WITH LOGIN;
    ```

   Note: Uncomment the line only if the "gitlab" role needs login privileges.

  * Exit the PostgreSQL terminal:
    ```sql
    \q
    ```

  * Exit the PostgreSQL user session:
    ```bash
    exit
    ```

These steps ensure that the data is imported into GitLab and appropriate privileges are granted to the PostgreSQL user.

Make sure to customize paths, usernames, and database names based on your specific setup.
# GitLab Upgrade Instructions

This repository contains instructions for upgrading GitLab to different versions using the RPM package manager on a CentOS-based system.

## Prerequisites :

- CentOS-based system
- Administrative privileges (sudo)

## Instructions

  * Create a file to skip auto-reconfigure:

    ```bash
    sudo touch /etc/gitlab/skip-auto-reconfigure
    ```

# GitLab CE Upgrade to Version 11.11.8:

    
    sudo yum install -y gitlab-ce-11.11.8
    

  * Perform PostgreSQL upgrade:

    ```bash
    sudo gitlab-ctl pg-upgrade
    ```

  * Reconfigure GitLab:

    ```bash
    sudo gitlab-ctl reconfigure
    ```

  * Run analyze_new_cluster.sh:

    ```bash
    #running analyze_new_cluster.sh
    Ctrl+Z
    ```

  * Deployment Down:

    ```bash
    #deployment down
    ```

# GitLab CE Upgrade to Version 12.0.12:

    sudo yum install -y gitlab-ce-12.0.12

  * Reconfigure GitLab:

    ```bash
    sudo gitlab-ctl reconfigure
    ```

# GitLab CE Upgrade to Version 12.1.17:
  
    sudo yum install -y gitlab-ce-12.1.17

  * Reconfigure GitLab:

    ```bash
    sudo gitlab-ctl reconfigure
    ```

# GitLab CE Upgrade to Version 12.10.14:

    sudo yum install -y gitlab-ce-12.10.14
  

  * Reconfigure GitLab:

    ```bash
    sudo gitlab-ctl reconfigure
    ```

Note: Make sure to adapt these instructions to your specific system and backup your data before performing any upgrades.
# Troubleshooting PG::UndefinedObject: ERROR: operator class "gin_trgm_ops" does not exist for access method "gin" error in GitLab, related to the missing PostgreSQL extension pg_trgm`.


## Instructions :

  * Install PostgreSQL 15 contrib package:

    ```bash
    sudo yum install -y postgresql15-contrib
    ```

  * Access the PostgreSQL database:

    ```bash
    psql -d gitlabhq_production -U postgres
    ```

  * Inside the PostgreSQL shell, create the `pg_trgm` extension:

    ```sql
    CREATE EXTENSION pg_trgm;
    ```

  * Restart PostgreSQL service:

    ```bash
    sudo gitlab-ctl restart postgresql
    sudo systemctl restart postgresql-15
    ```

  * Restart GitLab services:

    ```bash
    gitlab-ctl restart
    gitlab-ctl restart unicorn
    ```

  * Perform PostgreSQL upgrade (if needed):

    ```bash
    sudo gitlab-ctl pg-upgrade
    ```

  * Reconfigure GitLab:

    ```bash
    sudo gitlab-ctl reconfigure
    ```
# GitLab CE Upgrade to Version 13.0.14
  
    sudo yum install -y gitlab-ce-13.0.14
  * Restart GitLab services:
    ```
    sudo gitlab-ctl restart
   
  * Reconfigure GitLab:
    ```
    sudo gitlab-ctl reconfigure
# GitLab CE Upgrade to Version 13.1.11

    sudo yum install -y gitlab-ce-13.1.11
    
  * Reconfigure GitLab:
    ```bash
    sudo gitlab-ctl reconfigure
    ```
  * (Optional) Access the PostgreSQL database:

    ```bash
    psql -d gitlabhq_production -U postgres
    ```

  * (Optional) Inside the PostgreSQL shell, create the `btree_gist` extension:
    ```bash
    CREATE EXTENSION btree_gist;
    ```


# Ruby 2.7 Installation

These instructions will guide you through the process of installing Ruby 2.7 on your system using RVM (Ruby Version Manager).

# Prerequisites :

  * Make sure you have the following dependencies installed on your system:

    ```bash
    yum install gcc-c++ patch readline readline-devel zlib zlib-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison sqlite-devel
    ```

  ## RVM Installation

  * Install RVM by running the following commands:

    ```bash
    curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
    curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -
    curl -L get.rvm.io | bash -s stable
    ```

  * Load RVM into the current shell session:

    ```bash
    source /etc/profile.d/rvm.sh
    ```

  * Reload RVM to apply any changes:

    ```bash
    rvm reload
    ```

  * Run the RVM requirements to ensure all dependencies are met:

    ```bash
    rvm requirements run
    ```

## Ruby 2.7 Installation

  * List the available Ruby versions:

    ```bash
    rvm list known
    ```

  * Install Ruby 2.7:

    ```bash
    rvm install 2.7
    ```

  * List installed Ruby versions:

    ```bash
    rvm list
    ```

  * Set Ruby 2.7 as the default version:
    ```bash
    rvm use 2.7 --default
    ```

  * Now you have successfully installed Ruby 2.7 using RVM. Verify the installation by running:

    ```bash
    ruby -v
    ```
  * This should display the installed Ruby version.

# GitLab CE Upgrade to Version 13.0.14
### Step 1: *
  ```bash
  sudo yum install -y gitlab-ce-13.8.8
  ```

### Step 2: Access PostgreSQL and Modify Audit Events

  * Access the PostgreSQL database for GitLab:

    ```bash
    psql -d gitlabhq_production -U postgres
    ```

  * In the PostgreSQL shell, execute the following SQL command to change the owner of the `audit_events` table:

    ```sql
    ALTER TABLE audit_events OWNER TO postgres;
    ```

  * Exit the PostgreSQL shell.

### Step 3: Reconfigure GitLab

* Run the following command to reconfigure GitLab:

    ```bash
    sudo gitlab-ctl reconfigure
    ```

* This command will apply the changes and update the GitLab configuration.

## Notes :

Please note that the command `sudo yum install -y gitlab-ce-13.8.8` installs GitLab CE version 13.8.8. Adjust the version number according to your requirements.

**Important**: If you have previously used pgAdmin4 for database management, ensure to make the necessary adjustments in ``audit_events`` the command as specified in the note.


  * Remove the existing PostgreSQL data directory:

    ```bash
    sudo rm -rf /var/opt/gitlab/postgresql/data.11
    ```

  * Remove old PostgreSQL version file:

    ```bash
    sudo rm -f /var/opt/gitlab/postgresql-version.old
    ```

# GitLab CE Upgrade to Version 13.12.15
  ```bash
    sudo yum install -y gitlab-ce-13.12.15
  ```

  * Restart GitLab services:

    ```bash
    sudo gitlab-ctl restart
    ```

  * Reconfigure GitLab:

    ```bash
    sudo gitlab-ctl reconfigure
    ```

  * Web Hook Logs Update:

    **Note: Remove webhooks using pgadmin.**

  * Perform PostgreSQL upgrade:

    ```bash
    sudo gitlab-ctl pg-upgrade
    ```

### Important Warnings
- **Software Installation:** Ensure that installing GitLab CE version 13.12.15 is compatible with your system and won't cause conflicts with existing configurations.

Brief description of the project, its purpose, and any relevant information.

## Optional: GitLab Data Migration

If you want to migrate GitLab data to a different location, you can use the following optional commands:

  * Sync GitLab data to a new location:

    ```bash
    rsync -av /var/opt/gitlab/git-data/* /home/git_database/git-data/
    ```

   **Note: This step is optional and should be performed carefully. Ensure that the destination directory `/home/git_database/git-data/` is suitable for your setup.**

  * Change ownership of the new data directory:

    ```bash
    chown -R git:git /home/git_database/git-data/
    ```

  * Reconfigure GitLab to apply the changes:

    ```bash
    sudo gitlab-ctl reconfigure
    ```

### Important Considerations

- **Data Migration:** Performing data migration is an optional step and should be executed cautiously. Verify that the destination directory has sufficient space and the correct permissions.

- **Backup:** Before making any changes, it's advisable to create a backup of your GitLab data to avoid potential data loss.

# GitLab Storage Migration

This guide provides step-by-step instructions for performing a storage migration on your GitLab instance. Follow these steps carefully to ensure a smooth migration process.

## Prerequisites

- Access to the server where GitLab is installed.
- `sudo` privileges for executing commands.

## Steps

  * Open the GitLab Rails database console:

    ```bash
    sudo gitlab-rails dbconsole
    ```

  * Run the following SQL queries to nullify runner tokens:

    ```sql
    UPDATE projects SET runners_token = null, runners_token_encrypted = null;
    UPDATE namespaces SET runners_token = null, runners_token_encrypted = null;
    UPDATE application_settings SET runners_registration_token_encrypted = null;
    UPDATE ci_runners SET token = null, token_encrypted = null;
    ```

  * Migrate to hashed storage:

    ```bash
    sudo gitlab-rake gitlab:storage:migrate_to_hashed
    ```

  * Fix legacy hashed storage migration:

    ```bash
    # Download the fix script
    wget -O /tmp/fix-legacy-hashed-storage-migration.rb https://gitlab.com/snippets/2039252/raw

    # Run the script using GitLab Rails runner
    gitlab-rails runner /tmp/fix-legacy-hashed-storage-migration.rb
    ```

  * Re-run storage migration:

    ```bash
    sudo gitlab-rake gitlab:storage:migrate_to_hashed
    ```

  * List legacy projects:

    ```bash
    sudo gitlab-rake gitlab:storage:list_legacy_projects
    ```

  * List legacy attachments:

    ```bash
    sudo gitlab-rake gitlab:storage:list_legacy_attachments
    ```

  * Reconfigure GitLab:

    ```bash
    sudo gitlab-ctl reconfigure
    ```

## Conclusion

Your GitLab storage migration is now complete. Ensure that you review the output of each command for any errors and address them accordingly.

# GitLab Puma Configuration (Optional if you not able to upgrade 14.0.12)

## Introduction

This guide provides instructions on configuring GitLab with Puma as the web server. Puma is a concurrent web server that can handle multiple requests simultaneously, providing improved performance.

## Configuration Steps

  * Open the GitLab configuration file using a text editor. In this example, we use nano:

    ```bash
    sudo nano /etc/gitlab/gitlab.rb
    ```

  * Locate the section related to Unicorn and comment it out. Uncomment the corresponding section for Puma. Here's a snippet:

    ```ruby
    #nano /etc/gitlab/gitlab.rb

    #unicornComment out the part , and vice versa, pumauncomment the part.

    # ...

    ### Advanced settings
    puma['enable'] = true
    puma['ha'] = false
    puma['worker_timeout'] = 60
    puma['worker_processes'] = 2
    puma['min_threads'] = 4
    puma['max_threads'] = 4

    puma['listen'] = '127.0.0.1'
    puma['port'] = 8080
    puma['socket'] = '/var/opt/gitlab/gitlab-rails/sockets/gitlab.socket'
    puma['pidfile'] = '/opt/gitlab/var/puma/puma.pid'
    puma['state_path'] = '/opt/gitlab/var/puma/puma.state'

    # ...

    # Note: Undo these settings after upgrading to version 14.0.12
    ```

  * Save the changes and exit the text editor.

## Important Note

After upgrading GitLab to version 14.0.12, it is crucial to undo the Puma configuration settings and revert to the default configuration. Failing to do so may lead to issues. Use the same steps, uncommenting the Unicorn s


# GitLab CE Upgrade to Version 14.0.12

```bash
sudo yum install -y gitlab-ce-14.0.12
sudo gitlab-ctl restart
sudo gitlab-ctl reconfigure
```

# GitLab CE Upgrade to Version 14.3.6

  ```bash
    sudo yum install -y gitlab-ce-14.3.6
    sudo gitlab-ctl restart
    sudo gitlab-ctl reconfigure
  ```

## Background Migrations

  * Follow these steps as per the requests during reconfigure:

    ```bash
    sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,push_event_payloads,event_id,'[["event_id"]\, ["event_id_convert_to_bigint"]]']
    sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,events,id,'[["id"]\, ["id_convert_to_bigint"]]']
    sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,ci_stages,id,'[["id"]\, ["id_convert_to_bigint"]]']
    sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,taggings,id,'[["id"\, "taggable_id"]\, ["id_convert_to_bigint"\, "taggable_id_convert_to_bigint"]]']
    sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,ci_builds,id,'[["id"\, "stage_id"]\, ["id_convert_to_bigint"\, "stage_id_convert_to_bigint"]]']
    sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,ci_job_artifacts,id,'[["id"\, "job_id"]\, ["id_convert_to_bigint"\, "job_id_convert_to_bigint"]]']
    sudo gitlab-rake gitlab:background_migrations:finalize[CopyColumnUsingBackgroundMigrationJob,ci_builds_metadata,id,'[["id"]\, ["id_convert_to_bigint"]]']


  * After the background migrations, reconfigure GitLab:

    ```bash
    sudo gitlab-ctl reconfigure
    ```
# GitLab CE Upgrade to Version 14.9.5

```bash
sudo yum install -y gitlab-ce-14.9.5
sudo gitlab-ctl restart
sudo gitlab-ctl reconfigure
```

# GitLab CE Upgrade to Version 14.10.5

  ```bash
    sudo yum install -y gitlab-ce-14.10.5
    sudo gitlab-ctl restart
    sudo gitlab-ctl reconfigure
  ```
# Elasticsearch Installation Guide

This guide provides step-by-step instructions on how to install Elasticsearch on a Linux system using RPM packages.

## Prerequisites

Make sure you have the following prerequisites installed on your system:

- **wget**: To download files from the internet.
- **shasum**: To verify the integrity of the downloaded files.
- **sudo**: To perform administrative tasks.

## Installation Steps

1. **Download Elasticsearch RPM Package:**

    ```bash
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.8.0-x86_64.rpm
    ```

2. **Download and Verify SHA512 Checksum:**

    ```bash
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.8.0-x86_64.rpm.sha512
    shasum -a 512 -c elasticsearch-8.8.0-x86_64.rpm.sha512
    ```

    Ensure that the checksum matches before proceeding to the next step.

3. **Install Elasticsearch:**

    ```bash
    sudo rpm --install elasticsearch-8.8.0-x86_64.rpm
    ```

4. **Start Elasticsearch Service:**

    ```bash
    sudo systemctl start elasticsearch.service
    ```

## Verify Installation

To verify that Elasticsearch is running, you can check the status of the service:

```bash
sudo systemctl status elasticsearch.service
```
# GitLab CE Upgrade to Version 15.0.5

  ```bash
    sudo yum install -y gitlab-ce-15.0.5
    sudo gitlab-ctl restart
    sudo gitlab-ctl reconfigure
  ```
# Optional Cleanup Commands

If you need to perform cleanup tasks related to GitLab's PostgreSQL data, you can use the following optional commands. Be cautious when executing these commands, as they can result in data loss.

## Remove PostgreSQL Data (Optional)

**Note:** Only execute these commands if you are sure about the consequences, as they involve deleting data.

1. **Remove PostgreSQL Data Directory:**

    ```bash
    sudo rm -rf /var/opt/gitlab/postgresql/data.12
    ```

    This command will recursively remove the PostgreSQL data directory for GitLab. Ensure that you have a backup if necessary.

2. **Remove Old PostgreSQL Version File:**

    ```bash
    sudo rm -f /var/opt/gitlab/postgresql-version.old
    ```

    If there is an old PostgreSQL version file, this command will remove it. Make sure it's safe to delete this file based on your system configuration.

## Caution
Executing these commands will result in data deletion, and it is recommended to perform these actions only if you are confident about the necessity. Always have backups in place before making significant changes to your system.

# GitLab CE Upgrade to Version 15.4.6

  ```bash
    sudo yum install -y gitlab-ce-15.4.6
    sudo gitlab-ctl restart
    sudo gitlab-ctl reconfigure
  ```
# GitLab CE Upgrade to Version 15.11.13

  ```bash
    sudo yum install -y gitlab-ce-15.11.13
    sudo gitlab-ctl restart
    sudo gitlab-ctl reconfigure
  ```

# GitLab PostgreSQL Database Maintenance

This section provides commands for performing maintenance tasks related to the GitLab PostgreSQL database.

## View OAuth Access Tokens

  * To view OAuth access tokens in the `gitlabhq_production` database:

    ```bash
    psql -d gitlabhq_production -U postgres
    SELECT * FROM oauth_access_tokens WHERE expires_in IS NULL;
    ```

  * This command connects to the PostgreSQL database and retrieves all rows from the `oauth_access_tokens` table where the `expires_in` column is `NULL`.

## Update OAuth Access Token Expiry
  * To update the expiry time for a specific OAuth access token in the `gitlabhq_production` database:

    ```bash
    psql -d gitlabhq_production -U postgres
    UPDATE oauth_access_tokens SET expires_in = '86400' WHERE id = [ID];
    ```

  * The expiry time is set to 86400 seconds (24 hours).

## Reconfigure GitLab

  * After making changes to the database, it's recommended to reconfigure GitLab:

    ```bash
    sudo gitlab-ctl reconfigure
    ```

  * This command ensures that the changes are applied and the GitLab instance is properly configured.

# GitLab CE Upgrade to Version 16.2.1

  ```bash
    sudo yum install -y gitlab-ce-16.2.1
    sudo gitlab-ctl reconfigure
  ```

# Note: 
Once you reach v16, you will easily upgrade to upcoming versions.


## 🔗 Links
[![portfolio](https://img.shields.io/badge/my_portfolio-000?style=for-the-badge&logo=ko-fi&logoColor=white)](https://github.com/Hardik-Mistri)
[![linkedin](https://img.shields.io/badge/linkedin-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/hardikmistry299/)
[![twitter](https://img.shields.io/badge/twitter-1DA1F2?style=for-the-badge&logo=twitter&logoColor=white)](https://twitter.com/HardikM68728043)

