# Examples

## CLI Commands

### `start`

Run the `start` command to launch the daemon process that automatically initializes and unseals a Vault server:

```shell
$ vault-init start \
  --vault-addr "http://127.0.0.1:8200" \
  --local-encryption-secret-key "FjaUCqqTIorGTe1Z86rs2YfkRgQ6iIgo" \
  --postgres-storage-connection-url "postgres://vault:vault@127.0.0.1:5432/vault?sslmode=disable"
```

### `show`

Run the `show` command to fetch and decrypt the root token and unseal keys generated by Vault during the initialization process:

```shell
$ vault-init show \
  --local-encryption-secret-key "FjaUCqqTIorGTe1Z86rs2YfkRgQ6iIgo" \
  --postgres-storage-connection-url "postgres://vault:vault@127.0.0.1:5432/vault?sslmode=disable"
```

## Docker

See [`docker-compose.yaml`](../docker-compose.yaml) for an example on using the `vault-init` Docker image to initialize and unseal a Vault container running within the same Docker environment.

## Kubernetes

`vault-init` is a natural sidecar container for a Vault deployment on Kubernetes. An example of running it as a sidecar container can be found under [`docs/kubernetes`](kubernetes). Note that this is just for illustration purposes and is not a production-ready setup.

You will require a Kubernetes cluster to run the example:

1. Deploy a PostgreSQL instance into your cluster:

   ```shell
   $ kubectl apply -f kubernetes/example-postgres.yaml
   ```

2. Exec into the `example-postgres` Pod and create the database table required by `vault-init`:

   ```shell
   $ kubectl exec -it example-postgres-0 -- psql -U example
   $ psql > CREATE TABLE vault_init_data (
      encryption_type TEXT,
      encryption_version TEXT,
      root_token TEXT,
      unseal_keys TEXT[],
      created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
   );
   ```

3. Deploy Vault with a `vault-init` sidecar container into your cluster:

   ```shell
   $ kubectl apply -f kubernetes/example.yaml
   ```

4. Tail the logs of the `vault-init` container and watch it initialize and unseal Vault!

   ```shell
   $ kubectl logs -f example-0 vault-init
   ```

5. To clean up:

   ```shell
   $ kubectl delete -f kubernetes
   ```