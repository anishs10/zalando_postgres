# zalando_postgres

The Postgres Operator delivers an easy to run highly-available PostgreSQL clusters on Kubernetes (K8s) powered by Patroni. It is configured only through Postgres manifests (CRDs) to ease integration into automated CI/CD pipelines with no access to Kubernetes API directly, promoting infrastructure as code vs manual operations.

**Operator features**

1. Rolling updates on Postgres cluster changes, incl. quick minor version updates
2. Live volume resize without pod restarts (AWS EBS, PVC)
3. Database connection pooling with PGBouncer
4. Support fast in place major version upgrade. Supports global upgrade of all clusters.
5. Restore and cloning Postgres clusters on AWS, GCS and Azure
6. Additionally logical backups to S3 or GCS bucket can be configured
7. Standby cluster from S3 or GCS WAL archive
8. Configurable for non-cloud environments
9. Basic credential and user management on K8s, eases application deployments
10. Support for custom TLS certificates
11. UI to create and edit Postgres cluster manifests
12. Compatible with OpenShift


**There are top-level keys, containing both leaf keys and groups.**

**enable_crd_registration** Instruct the operator to create/update the CRDs. If disabled the operator will rely on the CRDs being managed separately. The default is true.

**enable_crd_validation deprecated:** toggles if the operator will create or update CRDs with OpenAPI v3 schema validation The default is true. false will be ignored, since apiextensions.io/v1 requires a structural schema definition.

**crd_categories** The operator will register CRDs in the all category by default so that they will be returned by a kubectl get all call. You are free to change categories or leave them empty.

**enable_lazy_spilo**_upgrade Instruct operator to update only the statefulsets with new images (Spilo and InitContainers) without immediately doing the rolling update. The assumption is pods will be re-started later with new images, for example due to the node rotation. The default is false.

**enable_pgversion_env_va**r With newer versions of Spilo, it is preferable to use PGVERSION pod environment variable instead of the setting postgresql.bin_dir in the SPILO_CONFIGURATION env variable. When this option is true, the operator sets PGVERSION and omits postgresql.bin_dir from SPILO_CONFIGURATION. When false, the postgresql.bin_dir is set. This setting takes precedence over PGVERSION; see PR 222 in Spilo. The default is true.

**enable_spilo_wal_path_compat** enables backwards compatible path between Spilo 12 and Spilo 13+ images. The default is false.

**enable_team_id_clustername_prefix** To lower the risk of name clashes between clusters of different teams you can turn on this flag and the operator will sync only clusters where the name starts with the teamId (from spec) plus -. Default is false.

**etcd_host** Etcd connection string for Patroni defined as host:port. Not required when Patroni native Kubernetes support is used. The default is empty (use Kubernetes-native DCS).

**kubernetes_use_configmaps** Select if setup uses endpoints (default), or configmaps to manage leader when DCS is kubernetes (not etcd or similar). In OpenShift it is not possible to use endpoints option, and configmaps is required. By default, kubernetes_use_configmaps: false, meaning endpoints will be used.

**docker_image** Spilo Docker image for Postgres instances. For production, don't rely on the default image, as it might be not the most up-to-date one. Instead, build your own Spilo image from the github repository.

**sidecar_docker_images deprecated**: use sidecars instead. A map of sidecar names to Docker images to run with Spilo. In case of the name conflict with the definition in the cluster manifest the cluster-specific one is preferred.

**sidecars** a list of sidecars to run with Spilo, for any cluster (i.e. globally defined sidecars). Each item in the list is of type Container. Globally defined sidecars can be overwritten by specifying a sidecar in the Postgres manifest with the same name. Note: This field is not part of the schema validation. If the container specification is invalid, then the operator fails to create the statefulset.

**enable_shm_volume** Instruct operator to start any new database pod without limitations on shm memory. If this option is enabled, to the target database pod will be mounted a new tmpfs volume to remove shm memory limitation (see e.g. the docker issue). This option is global for an operator object, and can be overwritten by enableShmVolume parameter from Postgres manifest. The default is true.

**workers** number of working routines the operator spawns to process requests to create/update/delete/sync clusters concurrently. The default is 8.

**max_instances** operator will cap the number of instances in any managed Postgres cluster up to the value of this parameter. When -1 is specified, no limits are applied. The default is -1.

**min_instances** operator will run at least the number of instances for any given Postgres cluster equal to the value of this parameter. Standby clusters can still run with numberOfInstances: 1 as this is the recommended setup. When -1 is specified for min_instances, no limits are applied. The default is -1.

**ignore_instance_limits_annotation_key** for some clusters it might be required to scale beyond the limits that can be configured with min_instances and max_instances options. You can define an annotation key that can be used as a toggle in cluster manifests to ignore globally configured instance limits. The default is empty.

**resync_period** period between consecutive sync requests. The default is 30m.

**repair_period** period between consecutive repair requests. The default is 5m.

**set_memory_request_to_limit** Set memory_request to memory_limit for all Postgres clusters (the default value is also increased but configured max_memory_request can not be bypassed). This prevents certain cases of memory overcommitment at the cost of overprovisioning memory and potential scheduling problems for containers with high memory limits due to the lack of memory on Kubernetes cluster nodes. This affects all containers created by the operator (Postgres, connection pooler, logical backup, scalyr sidecar, and other sidecars except sidecars defined in the operator configuration); to set resources for the operator's own container, change the operator deployment manually. The default is false.

**Postgres users:**

Parameters describing Postgres users. In a CRD-configuration, they are grouped under the users key.

**super_username **Postgres superuser name to be created by initdb. The default is postgres.

**replication_username** Postgres username used for replication between instances. The default is standby.

**additional_owner_roles** Specifies database roles that will be granted to all database owners. Owners can then use SET ROLE to obtain privileges of these roles to e.g. create or update functionality from extensions as part of a migration script. One such role can be cron_admin which is provided by the Spilo docker image to set up cron jobs inside the postgres database. In general, roles listed here should be preconfigured in the docker image and already exist in the database cluster on startup. Otherwise, syncing roles will return an error on each cluster sync process. Alternatively, you have to create the role and do the GRANT manually. Note, the operator will not allow additional owner roles to be members of database owners because it should be vice versa. If the operator cannot set up the correct membership it tries to revoke all additional owner roles from database owners. Default is empty.

**enable_password_rotation** For all LOGIN roles that are not database owners the operator can rotate credentials in the corresponding K8s secrets by replacing the username and password. This means, new users will be added on each rotation inheriting all priviliges from the original roles. The rotation date (in YYMMDD format) is appended to the names of the new user. The timestamp of the next rotation is written to the secret. The default is false.

**password_rotation_interval** If password rotation is enabled (either from config or cluster manifest) the interval can be configured with this parameter. The measure is in days which means daily rotation (1) is the most frequent interval possible. Default is 90.

**password_rotation_user_retention** To avoid an ever growing amount of new users due to password rotation the operator will remove the created users again after a certain amount of days has passed. The number can be configured with this parameter. However, the operator will check that the retention policy is at least twice as long as the rotation interval and update to this minimum in case it is not. Default is 180.
