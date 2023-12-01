# mongodb-relational-migrator

1 - docker-compose up -d

create table customers(

	id serial primary KEY,
	name text,
	age numeric
)


insert into customers (name, age) values ('Henrique', 22)

2 - GRANT USAGE ON SCHEMA "public" TO postgres;
GRANT SELECT ON TABLE "public"."customers" to postgres;
GRANT CREATE ON DATABASE "relational_migration" to postgres;

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

GRANT REPLICATION_GROUP TO postgres;

ALTER TABLE "public"."customers" OWNER TO REPLICATION_GROUP;

DROP PUBLICATION IF EXISTS "migrator_relational_migration_publication";

CREATE PUBLICATION "migrator_relational_migration_publication" FOR TABLE "public"."customers";
