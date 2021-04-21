# Installing Anchore on Kubernetes

Both Anchore Engine and Enterprise software are distributed as container images, and can run in a Kubernetes cluster. 

## Helm Chart

The recommended way to run Anchore on Kubernetes is via the [Helm chart](). This will install and configure all the necessary components to run Anchore. The Helm chart can be configured to deploy Anchore Engine and Anchore Enterprise in just a few minutes. 

While the Helm chart exposes many configurations and automatically bootstraps the system, it **does not automatically operate Anchore**. Users are still responsible for learning how to monitor, backup, upgrade, manage, etc. the Anchore software. 

If you deploy Anchore Engine, the Helm chart has no required configuration, and will install Anchore Engine with defaults out of the box. 

With Anchore Enterprise, some additional configuration is required to successfully install the software. 

**Note**: Prior to going to production, it is highly recommended that you understand the architecture, and learn the configuration options. 

### Anchore Enterprise

The sections below will focus on deployment of Anchore Enterprise in a production environment. 

**Deployment Warning**: By default, the chart will install a non-production ready deployment of the software. This provides a less complicated out-of-the-box experience for new users, but will likely lead to problems in large scale production environments. Additionally, there are recommended security configuration options in the chart which are also disabled by default. 

#### Configuration Options

#### Storage

##### Anchore Database

Anchore Enterprise requires a PostgreSQL database (>=9.6) to operate. **Production Recommendation**: Run this database externally via a service such as Amazon RDS. 

##### Anchore Feeds Database

Anchore Enterprise provides an on-premise feeds service which is supported by a PostgreSQL database (>=9.6). **Production Recommendation**: Run this database externally via a service such as Amazon RDS.

###### Database Sizing

TODO

###### Database tuning

While the Anchore Feeds Database remains relatively the same size, the main Anchore Database will grow over time. Anchore provides systems to facilitate image management operations and storage over time. 

###### Anchore Client Connections to the Anchore Database

Every Anchore "Core" services connects to the database with a default: `connectionPoolSize: 30` and a `connectionPoolMaxOverflow: 100`. These settings on the Anchore services side control how many client connections each service can make concurrently. On the Postgres side, max_connections control how many client total can connect at once. 

**Production Recommendation:** Check the max_connections in Postgres via: `SHOW max_connections` prior to deployment of Anchore. The `max_connections` output should be (at minimum) greater than the total `connectionPoolSize` for all services.
- 30 Anchore services * 30 connectionPoolSize = 900 max_connections or greater. 
- To accommodate the burst `connectionPoolMaxOverflow: 100`. 30 Anchore services * 100 = 3000 max_connections. 

As long as your database has enough resources to handle incoming connections then the Anchore service pool won't bottleneck.

**Example Configuration: Anchore Database**

```
# Anchore engine has a dependency on Postgresql, configure here
postgresql:
  # To use an external DB or Google CloudSQL in GKE, uncomment & set 'enabled: false'
  # externalEndpoint, postgresUser, postgresPassword & postgresDatabase are required values for external postgres
  enabled: false
  postgresUser: anchoreengine
  postgresPassword: anchore-postgres,123
  postgresDatabase: anchore

  # Specify an external (already existing) postgres deployment for use.
  # Set to the host and port. eg. mypostgres.myserver.io:5432
  externalEndpoint: external-database-endpoint:5432

  # Configure size of the persistent volume used with helm managed chart.
  # This should be commented out if using an external endpoint.
  # persistence:
  #  resourcePolicy: nil
  #  size: 20Gi
```
##### Object Storage



