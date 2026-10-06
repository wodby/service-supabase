# Supabase on Wodby

What Wodby sets up for a self-hosted Supabase project. Check it before generating API keys, configuring components one by one, or pointing an application at a component directly.

## What runs

One Supabase project per environment, from the upstream self-hosted component images: a gateway (Envoy), Auth, REST (PostgREST), Realtime, Storage with imgproxy, postgres-meta and Studio, one instance each. Edge Functions, Supavisor and analytics are not part of this service, so Studio features that need them are unavailable.

## How it is reached

- Everything goes through the gateway: the service's endpoint `http`, port `8000`. Inside the environment the host is this app service's name.
- The public Supabase URL is the service's primary URL: it follows the service's route and is not set by hand.

## Keys and passwords

Generated once per environment as tokens of this service and kept across restarts and upgrades:

- `publishable_key`: for client applications, together with the public URL.
- `secret_key`: for trusted server code only. It bypasses row level security and must not reach a browser build.
- `dashboard_password`: the Studio password. The Studio user name is `supabase`.
- `signing_seed`, `secret_key_base`, `realtime_db_enc_key`, `pg_meta_crypto_key`: internal signing and encryption material. Data encrypted with them cannot be read after they change.

Do not generate Supabase keys by hand or copy keys from another project.

## Database link

The required `db` link accepts only the Supabase PostgreSQL service. Through it this service receives the database host and port, the database password and the JWT secret, which belong to the database service. The database is always `postgres`.

## Settings and integrations

Configuration is changed on the service, not in component files:

- Settings: `site_url` (required), `redirect_urls`, `disable_signup`, `db_schemas` (schemas exposed by the API), `smtp_sender_email` (required), `smtp_sender_name`, `storage_bucket`, `s3_endpoint`.
- Integration `smtp` (required) sends the authentication email.
- Integration `s3` (optional) switches Storage to an S3 bucket: it sets `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `REGION` and `STORAGE_BACKEND`. The bucket named in `storage_bucket` must already exist; `s3_endpoint` is for storage other than AWS. Objects already on the volume are not moved.

A new redirect target, such as a custom domain of the application, has to be added to `redirect_urls`.

## Data and backups

- Volume `storage`: objects of the filesystem storage backend. Volume `studio`: Studio snippets.
- Backups `objects` (storage volume) and `snippets` (Studio volume). An S3 bucket is not backed up by Wodby.
- Table data, users and storage metadata are in the linked database and are backed up there. A consistent recovery point needs the database backup, the objects and this service's tokens together.
