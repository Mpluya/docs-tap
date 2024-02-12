# Configure Artifact Metadata Repository

This topic tells you how to configure Artifact Metadata Repository (AMR).

## <a id='amr-observer'></a> AMR Observer

You can obtain the Tanzu Application Platform values schema by running:

```console
tanzu package available get amr-observer.apps.tanzu.vmware.com/${VERSION} --values-schema --namespace tap-install
```

The following example is an AMR Observer configuration, located under the `amr` key in the
Tanzu Application Platform values file:

```yaml
amr:
  observer:
    location: |
      labels:
      - key: environment
        value: prod
    resync_period: "10h"
    ca_cert_data: |
      -----BEGIN CERTIFICATE-----
      Custom CA certificate for AMR CloudEvent Handler's HTTPProxy with custom TLS certs
      -----END CERTIFICATE-----
    cloudevent_handler:
      endpoint: "https://amr-cloudevent-handler.DOMAIN"
      liveness_period_seconds: 10
    auth:
      kubernetes_service_accounts:
        enabled: true
        autoconfigured: true
        secret:
          ref: "amr-observer-edit-token"
          value: ""
    deployed_through_tmc: false
    max_concurrent_reconciles:
      image_vulnerability_scans: 1
```

Where `DOMAIN` is the domain you want to target.

Configuration options:

- `amr.observer.location`
  - Default: ""
  - Location is the multiline string configuration for the location content.
  - The YAML string can contain a single field:
    - `labels`: Consists of an array for key and value pairing. Useful for adding searchable and identifiable metadata. For enabling DORA functionality, it is important to have a label named `env`.
    For more information, see [DORA metrics in Tanzu Developer Portal](../../tap-gui/plugins/dora.hbs.md).

- `amr.observer.resync_period`
  - Default: "10h"
  - `resync_period` decides the minimum frequency at which watched resources reconcile. A lower period corrects entropy more quickly, but reduce responsiveness to change if there are many watched resources. Change this value only if you know what you are doing. Defaults to 10 hours if unset.

- `amr.observer.ca_cert_data` or `shared.ca_cert_data`
  - Default: ""
  - `ca_cert_data` adds certificates to the truststore that amr-observer uses.

    ```console
    kubectl -n metadata-store get secrets/amr-cloudevent-handler-ingress-cert -o jsonpath='{.data."crt.ca"}' | base64 -d
    ```

- `amr.observer.cloudevent_handler.endpoint`
  - Default: `http://amr-cloudevent-handler.metadata-store.svc.cluster.local:80`
  - The URL of the AMR CloudEvent Handler endpoint.
  - On the view or full Tanzu Application Platform profile cluster, obtain the AMR CloudEvent Handler ingress address to configure this property:
    
    ```console
    kubectl -n metadata-store get httpproxies.projectcontour.io amr-cloudevent-handler-ingress -o jsonpath='{.spec.virtualhost.fqdn}'
    ```

  >**Note** Ensure that you set the correct protocol. If there is TLS, you must prepend `https://`. If there is no TLS, you must prepend `http://`.

- `amr.observer.cloudevent_handler.liveness_period_seconds`
  - Default: `10`
  - The period in seconds between executed health checks to the AMR CloudEvent Handler endpoint.

- `amr.observer.auth.kubernetes_service_accounts`
  - `.enable`
    - Default: `true`
    - Include an Authorization header when communicating with AMR CloudEvent Handler.
  - `.autoconfigure`
    - Default: `true`
    - Delegate creation of authentication token secret to the artifact metadata repository. Only applicable on Full and View clusters.
  - `.secret`
    - The secret with the access token for communicating with the cloudevent-handler
    - `.ref`
      - Default: ""
      - Secret name which contains the access token.
    - `.value`
      - Default: ""
      - Secret as a plain text string. This allows integrating with TMC secret imports.

- `amr.observer.deployed_through_tmc`
  - Default: `null`
  - Tanzu Application Platform multicluster deployment happens through Tanzu Mission Control when you set `deployed_through_tmc` to true.

- `amr.observer.max_concurrent_reconciles`
  - Configure maximum concurrent reconciles for controllers.
  - `.image_vulnerability_scans`
    - Default: `1`
    - Maximum concurrent reconciles for observing ImageVulnerabilityScans.

## <a id='amr-graphql'></a> AMR GraphQL

- `amr.graphql.auth.kubernetes_service_accounts``
  - `.enable`
    - Default: true
    - Enable authentication for artifact metadata repository GraphQL server. By default it is set to true.
  - `.autoconfigure`
    - Default: `true`
    - Delegate creation of authentication token secret to the artifact metadata repository. By default it is set to true.

## <a id='amr-cloudevent-handler'></a> AMR CloudEvent Handler

- `amr.cloudevent_handler.auth.kubernetes_service_accounts`
  - `.enable`
    - Default: true
    - Enable authentication and authorization for services accessing Artifact Metadata Repository.
  - `.autoconfigure`
    - Default: `true`
    - Delegate creation of authentication token secret to the artifact metadata repository. By default it is set to true.
