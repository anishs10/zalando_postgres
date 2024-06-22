# zalando_postgres

The Postgres Operator delivers an easy to run highly-available PostgreSQL clusters on Kubernetes (K8s) powered by Patroni. It is configured only through Postgres manifests (CRDs) to ease integration into automated CI/CD pipelines with no access to Kubernetes API directly, promoting infrastructure as code vs manual operations.

Operator features

Rolling updates on Postgres cluster changes, incl. quick minor version updates
Live volume resize without pod restarts (AWS EBS, PVC)
Database connection pooling with PGBouncer
Support fast in place major version upgrade. Supports global upgrade of all clusters.
Restore and cloning Postgres clusters on AWS, GCS and Azure
Additionally logical backups to S3 or GCS bucket can be configured
Standby cluster from S3 or GCS WAL archive
Configurable for non-cloud environments
Basic credential and user management on K8s, eases application deployments
Support for custom TLS certificates
UI to create and edit Postgres cluster manifests
Compatible with OpenShift
