# docker-compose-secrets-envvars

Use Docker Compose v2 with Secrets and Environment Variables for a more secure deployment strategy.

Pass your Environment Variables as a Docker Secret and read the values from the respective files.

## Common Ways: Using `environment` / `env_file`

__CASE 1__

```yaml
service:
  alpine-test:
    image: alpine:latest
    command: sleep 3600
    environment:
      - TEST_CREDENTIALS=supersecret
```

__CASE 2__

The same credentials can be save in a `.env` file

```env
TEST_CREDENTIALS=supersecret
```

```yaml
services:
  alpine-test:
    image: alpine:latest
    command: sleep 3600
    environment:
      - TEST_CREDENTIALS=${TEST_CREDENTIALS} # from .env file
```

__CASE 3__

A dedicated environment file e.g. `test.env`:

```yaml
services:
  alpine-test:
    image: alpine:latest
    commmand: sleep 3600
    env_file:
      - test.env
```

### Caveats

The environment variable is still visible in the container's `env` or when performing the following:

```bash
docker inspect -f "{{ .Config.Env }}" <container_name / container_id>
```

OR

```bash
docker compose exec alpine-test env
```

This MAY or MAY NOT be desirable for certain deployments.

## Docker Secrets with Environment Variables

1. place the value in `.env` file

    ```
    TEST_CREDENTIALS=supersecret
    ```
2. add the `secrets` block in the `docker-compose.yml` as follows:

    ```yaml
    services:
      # snip
    secrets:
      test-credentials: # name of the secret (lower case)
        environment: TEST_CREDENTIALS # use the value from the .env (or even `export TEST_CREDENTIALS` works)
    ```

3. use the defined secret in the service:

    ```yaml
    services:
      alpine-test:
        image: alpine:latest
        command: sleep 3600
        secrets:
          - test-credentials
    secrets:
      # snip
    ```
Docker will create `/run/secrets/test-credentials` file within the container which can they be used by your App.

### Benefits

Plain-text Values of environment values will not appear directly upon:

```bash
docker inspect -f "{{ .Config.Env }}" <container hash / container name>
```

Nor will the values appear in:

    docker compose exec alpine-test env


### Caveats

- Your App might require file read logic in order to read credentials
- User ID / Group ID mismatch may cause problems for read permissions (can be mitigated by using `user` in compose file)
- credentials still in plain-text when in file

## Setup / Requirements

Tests performed on:

### OS

Manjaro Linux 

### Docker Version

Client and Server Engine __v23.0.1__

### Docker Compose Version

```
Docker Compose version 2.16.0
```

## Source

[Docker Blog New Compose v2 and v1 deprecation Post / Additional Updates Section](https://www.docker.com/blog/new-docker-compose-v2-and-v1-deprecation/)
