# Variable Scoping

DevOps variables provide a way to store and reuse values with our Docker containers, ultimately used by our Docker image hooks to customize configurations.

It's important to understand the different levels at which variables can be set and how and where you should use them. This diagram shows the different scopes in which variables can be set and applied: 

![Variable Scoping](images/variableScoping-1.png)

Generally, you'll set variables having an orchestration scope. 

## Image scope 

Variables having an image scope are assigned using the values set for the Docker Image (for example, from Dockerfiles). These variables are often set as defaults, allowing narrower scopes to override them.

To see the default environment variables available with any Docker image, you can enter:

  ```shell
  docker run pingidentity/<product-image>:<tag> env | sort
  ```

  Where \<product-image> is the name of one of our products, and \<tag> is the release tag (such as, `edge`).

See our [Docker images reference](https://pingidentity-devops.gitbook.io/devops/dockerimagesref) for the environment variables available or each product, as well as those available for all products (PingBase). 

## Orchestration scope

Variables having orchestration scope are assigned at the orchestration layer.  Typically, these are environment variables set using Docker commands, or Docker Compose or Kubernetes YAML configuration files. For example:

* Using docker run with --env:

  ```shell
  docker run --env SCOPE=env \
    pingidentity/pingdirectory:edge env | sort
  ```

* Using docker run with --env-file:

  ```shell
  echo "SCOPE=env-file"  > /tmp/scope.properties

  docker run --env-file /tmp/scope.properties \
    pingidentity/pingdirectory:edge env | sort
  ```

* Using Docker Compose (docker-compose.yaml):

  ```yaml
  environment:
    - SCOPE=compose
      env_file:
    - /tmp/scope.properties
  ```

* Using Kubernetes (kustomize.yaml)

  ```yaml
  env:
    - name: SCOPE
      value: kubernetes
  ```

* Using Kubernetes configMapRef and secretRef (kustomize.yaml)

  ```yaml
  - envFrom:
    - configMapRef:
      name: kubernetes-variables
    - secretRef:
      name: kubernetes-secret
  ```

## Server Profile scope 

A property file provided by the server-profile repo.  Carefully consider
setting variables in this scope as can override Image/Orchestration 
scoped variables.

The following masthead can be used for your env_vars file to provide 
examples of setting variables and how they might override lower level
scoped variables.  It will also suppress a warning when processing
the env_vars file

    # .suppress-container-warning
    #
    # NOTICE: Settings in this file will override values set at the
    #         image or orchestraton layers of the container.  Examples
    #         include variables that are specific to this server profile.
    #
    # Options include:
    #
    # ALWAYS OVERRIDE the value in the container
    #   NAME=VAL        
    #
    # SET TO DEFAULT VALUE if not already set       
    #   export NAME=${NAME:=myDefaultValue}  # Sets to string of "myDefaultValue"
    #   export NAME=${NAME:-OTHER_VAR}       # Sets ot value of OTHER_VAR variable
    # 


## Container scope 
Any variables defined in the hook scripts.  Variables that need to be passed to other hook scripts can append to ${CONTAINER_ENV}, 
(defined as /opt/staging/.env).  This file will be sourced for every hook.

# Example Scoping

![Variable Scoping](images/variableScoping-2.png)