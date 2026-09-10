# Supabase service for Wodby

Run self-hosted Supabase Auth, REST, Realtime, Storage and Studio on Kubernetes using the upstream self-hosted/v0.8.1 images and the Wodby Supabase chart.

## Use this service

Use this service through the [Supabase stack](https://github.com/wodby/stack-supabase).

## Configuration

Connect a Supabase PostgreSQL service through the required `db` link. Supply a site URL, email sender address and SMTP integration. The public Supabase URL follows the Wodby service's canonical route. Update the application's redirect allowlist deliberately when adding a custom domain.

The service creates persistent per-environment tokens for API access, signing, Studio login and component encryption. The Studio username is `supabase`; reveal the `dashboard_password` token to obtain its password. Client applications use the public URL and `publishable_key`; trusted server applications can use `secret_key`. Server keys bypass ordinary user RLS restrictions and must never be included in a browser build.

The database service owns the database password and legacy JWT secret. The one-way link supplies these values to the application bundle. Matching token names on separate services are not a mechanism for sharing generated credentials.

SMTP uses the generic SMTP integration. Optional S3 storage uses a variable integration requiring `AWS_ACCESS_KEY_ID`, secret `AWS_SECRET_ACCESS_KEY`, and `AWS_REGION`. These are the variables read by the storage runtime. Set `storage_bucket` and, for non-AWS storage, `s3_endpoint`. The bucket must exist before use. Connecting this integration selects the S3 backend; migrating existing filesystem objects is a separate operation.

## Persistence and recovery

The `storage` volume contains filesystem-backed objects. The `studio` volume contains snippets. Database metadata and stored objects must be backed up as a coordinated recovery point with application writers stopped. The filesystem objects backup does not back up an external S3 bucket.

The linked database service uses the released `wodby/supabase-postgres:17-0.1.0` image. Its native import restores a checksummed backup into a fresh database volume, including the root encryption key. Restore the matching filesystem or S3 objects separately before resuming application writers.

Save all service tokens and the linked database tokens as recovery material. Preserve encryption keys when restoring data. Environment copying preserves token values: rotate client credentials deliberately after copying when isolation requires it, while retaining keys needed to read copied encrypted data.

The chart bootstrap job derives credentials from persistent source tokens. Routine restarts and upgrades retain public API keys and signing keys. Rotate signing seeds separately from API keys; encryption-key rotation requires a supported data migration.

## Initial support boundary

This service represents one self-hosted Supabase project per Wodby app environment. It uses one replica per component. Edge Functions, Supavisor, Supabase analytics, MCP access and database HA are not included. Studio features relying on omitted services are unavailable. Supabase Cloud organization management, branching and managed PITR are separate products.

Upgrade the tested component bundle together. PostgreSQL-major upgrades and migration from an existing Supabase deployment require a separately validated procedure. A Helm rollback cannot reverse database migrations.

## Maintain a custom version

Fork this repository, update the manifest and referenced configuration, and validate the complete Supabase bundle before importing your service.
