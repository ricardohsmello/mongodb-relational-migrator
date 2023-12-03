# Relational Migrator

## Video demonstration

[![Sample](https://img.youtube.com/vi/nvKm4PCEmC0/0.jpg)](https://www.youtube.com/watch?v=nvKm4PCEmC0)


## Overview

This repository contains the Relational Migrator, a tool designed for performing relational data migrations and synchronization. This README provides instructions on how to set up the environment using Docker and execute a data synchronization job.

## Getting Started

### Prerequisites

- Docker installed on your machine ([Docker Installation Guide](https://docs.docker.com/get-docker/))
- Docker Compose installed on your machine ([Docker Compose Installation Guide](https://docs.docker.com/compose/install/))

### Installation

1. **Clone this repository:**

    ```bash
    git clone https://github.com/ricardohsmello/mongodb-relational-migrator.git
    ```

2. **Navigate to the project directory:**

    ```bash
    cd relational-migrator
    ```

3. **Start the Docker containers:**

    ```bash
    docker-compose up -d
    ```

4. **Access the PostgreSQL database using a database client like DBeaver or psql, and create a sample table named `customers`:**

    ```sql
    CREATE TABLE customers (
        id SERIAL PRIMARY KEY,
        name TEXT,
        age NUMERIC
    );

    INSERT INTO customers (name, age) VALUES ('Henrique', 22);
    ```

    ...

5. **Provide necessary grants for CDC (Change Data Capture) functionality:**

    ```sql
    GRANT USAGE ON SCHEMA "public" TO postgres;
    GRANT SELECT ON TABLE "public"."customers" TO postgres;
    GRANT CREATE ON DATABASE "relational_migration" TO postgres;
    
    DO
        $do$
            BEGIN
                IF NOT EXISTS (
                    SELECT * FROM pg_catalog.pg_roles
                        WHERE  rolname = 'replication_group') THEN
                    CREATE ROLE REPLICATION_GROUP;
                ELSE
                    RAISE NOTICE 'Role "replication_group" already exists. Skipping.';
                END IF;
            END
        $do$;
    
    GRANT REPLICATION_GROUP TO postgres;
    ALTER TABLE "public"."customers" OWNER TO REPLICATION_GROUP;
    ```

6. **Create a publication for the `customers` table:**

    ```sql
    DROP PUBLICATION IF EXISTS "migrator_relational_migration_publication";
    CREATE PUBLICATION "migrator_relational_migration_publication" FOR TABLE "public"."customers";
    ALTER TABLE "public"."customers" REPLICA IDENTITY FULL;
    ```

7. **Update the `wal_level` parameter to 'logical':**

    ```bash
    docker exec -it <your_postgres_container_id> sed -i '/^#wal_level/s/^#//' /var/lib/postgresql/data/postgresql.conf
    docker exec -it <your_postgres_container_id> sed -i 's/wal_level = replica/wal_level = logical/' /var/lib/postgresql/data/postgresql.conf
    ```

    Replace `<your_postgres_container_id>` with the actual container ID.

8. **Restart the PostgreSQL container:**

    ```bash
    docker restart <your_postgres_container_id>
    ```

9. **Verify the `wal_level` value:**

    Connect to your PostgreSQL database using DBeaver or another database client and execute:

    ```sql
    SHOW wal_level;
    ```

    The result should be `logical`.

