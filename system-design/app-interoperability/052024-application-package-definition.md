# Proposal for updated application definition

## Simple Hello World Example

````yaml
apiVersion: margo.org/v1-alpha1
kind: ApplicationDescription
metadata:
  id: northstartida.hello.world
  name: Hello World
  description: A basic hello world application
  version: 1.0
  catalog:
    application:
      icon: ./resources/hw-logo.png
      tagline: Northstar Industrial Application's hello world application.
      descriptionLong: ./resources/description.md
      releaseNotes: ./resources/release-notes.md
      licenseFile: ./resources/license.pdf
      site: http://www.northstar-ida.com
    author:
      name: Roger Wilkershank
      email: rpwilkershank@northstar-ida.com
    organization:
      name: Northstar Industrial Applications
      site: http://northstar-ida.com
sources:
  - name: hello-world
    type: helm.v3
    properties:  
      repository: oci://northstarida.azurecr.io/charts/hello-world
      revision: 1.0.1
      wait: true
minimumResourceRequirements:
  cpu: 0.5
  memory: 16384
  storage:
    containers: 3788.8
    appStorage: 20480
properties:
  greeting:
    value: Hello
    targets:
    - pointer: /hello-world/env/APP_GREETING
  target:
    value: World
    targets:
    - pointer: /hello-world/env/APP_TARGET
configuration:
  sections:
    - name: General Settings
      settings:
        - property: greeting
          name: Greeting
          description: The greeting to use.
          inputType: text
          schema: requireText
        - property: target
          name: Greeting Target
          description: The target of the greeting.
          inputType: text
          schema: requireText
  schema:
    - name: requireText
      appliesTo: text
      maxLength: 45
      allowEmpty: false
````

## Complex Example

````yaml
apiVersion: margo.org/v1-alpha1
kind: ApplicationDescription
metadata:
  id: northstartida.digitron.orchestrator
  name: Digitron orchestrator
  description: The Digitron orchestrator application
  version: 1.2.1 
  catalog:
    application:
      icon: ./resources/ndo-logo.png
      tagline: Northstar Industrial Application's next-gen, AI driven, Digitron instrument orchestrator.
      descriptionLong: ./resources/description.md
      releaseNotes: ./resources/release-notes.md
      licenseFile: ./resources/license.pdf
      site: http://www.northstar-ida.com
    author:
      name: Roger Wilkershank
      email: rpwilkershank@northstar-ida.com
    organization:
      name: Northstar Industrial Applications
      site: http://northstar-ida.com
sources:
  - name: digitron-orchestrator
    type: helm.v3
    properties:
      repository: oci://northstarida.azurecr.io/charts/northstarida-digitron-orchestrator
      revision: 1.0.9
      wait: true
  - name: database-services
    type: helm.v3
    properties: 
      repository: oci://quay.io/charts/realtime-database-services
      revision: 2.3.7
      wait: true
  - name: digitron-orchestrator-docker
    type: docker-compose
    properties:
      packageLocation: https://northsitarida.com/digitron/docker/digitron-orchestrator.tar.gz
      keyLocation: https://northsitarida.com/digitron/docker/public-key.asc
minimumResourceRequirements:
  cpu: 0.5
  memory: 16384
  storage:
    containers: 3788.8
    appStorage: 20480
  customResource:
    - name: nvidia.com/gpu
      limit: 1
properties:
  storageClass:
    value: ${cluster.storageClass}
    targets:
      - pointer: /global/storageClassName
        sources: ["digitron-orchestrator", "database-services"]
  hostname:
    value: ${node.hostname}
    targets:
      - pointer: /edge_url
        prefix: https://
        sources: ["digitron-orchestrator"]
      - pointer: /global/ingress/tlsSecretName
        sources: ["digitron-orchestrator", "database-services"]
      - pointer: /global/ingress/host
        sources: ["digitron-orchestrator", "database-services"]
      - pointer: ENV.HOSTNAME
        sources: ["digitron-orchestrator-docker"]
  idpName:
    value: ${idp.name}
    targets:
      - pointer: /idp/name
        sources: ["digitron-orchestrator"]
      - pointer: ENV.IDP_NAME
        sources: ["digitron-orchestrator-docker"]
  idpProvider:
    value: ${idp.provider}
    targets:
      - pointer: /idp/provider
        sources: ["digitron-orchestrator"]
      - pointer: ENV.IDP_PROVIDER
        sources: ["digitron-orchestrator-docker"]
  idpClientId:
    value: ${idp.clientId}
    targets:
      - pointer: /idp/clientId
        sources: ["digitron-orchestrator"]
      - pointer: ENV.IDP_CLIENT_ID
        sources: ["digitron-orchestrator-docker"]
  idpUrl:
    value: ${idp.url}
    targets:
      - pointer: /idp/providerUrl
        sources: ["digitron-orchestrator"]
      - pointer: /idp/providerMetadata
        sources: ["digitron-orchestrator"]
        suffix: /.well-known/openid-configuration
      - pointer: ENV.IDP_URL
        sources: ["digitron-orchestrator-docker"]
  adminName:
    value: ${admin.fullname}
    targets:
      - pointer: /administrator/name
        sources: ["digitron-orchestrator"]
      - pointer: ENV.ADMIN_NAME
        sources: ["digitron-orchestrator-docker"]
  adminUpn:
    value: ${admin.username}
    targets:
      - pointer: /administrator/userPrincipalName
        sources: ["digitron-orchestrator"]
      - pointer: ENV.ADMIN_USERNAME
        sources: ["digitron-orchestrator-docker"]
  pollFrequency:
    value: 30
    targets: 
      - pointer: /settings/pollFrequency
        sources: ["digitron-orchestrator"]
      - pointer: ENV.POLL_FREQUENCY
        sources: ["digitron-orchestrator-docker"]
  siteId:
    targets:
      - pointer: /settings/siteId
        sources: ["digitron-orchestrator"]
      - pointer: ENV.SITE_ID
        sources: ["digitron-orchestrator-docker"]
  cpuLimit:
    targets:
      - pointer: /settings/limits/cpu
        sources: ["digitron-orchestrator"]
  memoryLimit:
    targets:
      - pointer: /settings/limits/memory
        sources: ["digitron-orchestrator"]
configuration:
  sections:
    - name: General
      settings:
        - property: pollFrequency
          name: Poll Frequency
          description: How often the service polls for updated data in seconds
          dataType: integer
          schema: pollRange
        - property: siteId
          name: Site Id
          description: Special identifier for the site (optional)
          dataType: string
          schema: optionalText
    - name: Identity Provider
      settings:
        - property: idpName
          name: Name
          description: The name of the Identity Provider to use
          dataType: string
          immutable: true
          schema: requiredText
        - property: idpProvider
          name: Provider
          description: Provider something something
          dataType: string
          immutable: true
          schema: requiredText
        - property: idpClientId
          name: Client ID
          description: The client id
          dataType: string
          immutable: true
          schema: requiredText
        - property: idpUrl
          name: Provider URL
          description: The url of the Identity Provider
          immutable: true
          dataType: string
          schema: url
    - name: Administrator
      settings:
        - property: adminName
          name: Presentation Name
          description: The presentation name of the administrator
          dataType: string
          schema: requiredText
        - property: adminUpn
          name: Principal Name
          description: The principal name of the administrator
          dataType: string
          schema: email
    - name: Resource Limits
      settings:
        - property: cpuLimit
          name: CPU Limit
          description: Maximum number of CPU cores to allow the application to consume
          dataType: double
          schema: cpuRange
        - property: memoryLimit
          name: Memory Limit
          description: Maximum number of memory to allow the application to consume
          dataType: integer
          schema: memoryRange
  schema:
    - name: requiredText
      appliesTo: string
      maxLength: 45
      allowEmpty: false
    - name: email
      allowEmpty: false
      regexMatch: .*@[a-z0-9.-]*
    - name: url
      allowEmpty: false
      regexMatch: ^(http(s):\/\/.)[-a-zA-Z0-9@:%._\+~#=]{2,256}\.[a-z]{2,6}\b([-a-zA-Z0-9@:%_\+.~#?&//=]*)$
    - name: pollRange
      minValue: 30
      maxValue: 360
      allowEmpty: false
    - name: optionalText  
      minLength: 5
      allowEmpty: true
    - name: cpuRange
      minValue: 0.5
      maxPrecision: 1
      allowEmpty: false
    - name: memoryRange
      minValue: 16384
      allowEmpty: false
````

## Proposed Changes

1. Updated the document format to be a little less like a custom resource definition (e.g., removing "spec") so it  is not confused with being a CRD.
2. Updated the metadata to provide a bit more organization.
   - added ID that can be used for things like helping to create unique namespaces
3. Changed to use `sources` with a `type` instead of having specific `helm-chart` and `docker-compose` sections
   - This approach is more extensible because we will be needing to add new deployment types over time (e.g. Helm v4, web assembly, etc.).
4. Updated the specification to be package-based (helm chart, docker-compose tarball) instead of allowing for loose files.
   - This is more secure because packages can be signed and loose files cannot
   - Also, makes it easier for the workload orchestration vendors and device vendors to manage
5. Updated the format to be able to have multiple helm charts (or docker-compose files)
   - This enables deploying different configurations without having to create a helm chart wrapper for each combination
   - This is better for using 3rd part helm charts as well because we don't have to create a wrapper around 3rd party helm chart we don't own
   - supporting multiple helm charts:
     - The order the helm chart is installed is the order the charts is listed in the application definition
     - The `wait` property, if `true`, instructs the device to wait for that helm chart to be installed before continuing to the next chart
       - We can make it optional for a device to support `false`
       - When it is true, the device should return an error status immediately upon failure and not try to install any subsequent charts
   - I'm not sure if having multiple docker-compose files makes sense but it can be supported using the same rules.
6. Added the `minimumResourceRequirements` section
   - Margo would need to define what these properties are and what the value units are so it is consistent. For example:
     - For CPU:
       - Unit: 1.0 = 100% of one core
       - DataType: Double,  
     - For Memory:  
       - Unit: 1 = 1 Mebibyte
       - DataType: Double
     - For storage:
       - Unit: 1 = 1 Mebibyte
       - DataType: Double
   - For custom resources:
     - The way this is defined and used in kubernetes is going to be changing so there may be a better way of handling this soon. We'll have to evaluate over the next few months. I'm not aware of any similar functionality in Docker.
     - For now we can use the syntax you have to use for claiming a custom resource in Kubernetes
7. Using `properties` instead of `parameters`
   - This term feels a bit bit more generic in intent and adds some flexibility
8. Updates to the `targets`
   - using `pointer` instead of `key`
     - Using [RFC6901 (JSON Pointer) specification](https://datatracker.ietf.org/doc/html/rfc6901) notation for the targets
       - It's called "JSON Pointer" but it can be used with other hierarchical formats like YAML
   - using `sources` instead of `appliesTo`
   - It may not be something we want to include with Margo but we could also have the ability to specify a `prefix` and/or `suffix` to append to the value provided for the parameter.
     - This could be handled in the helm chart itself but there may be cases where the helm chart isn't ideal and this would be helpful
9. Splitting out the `properties` from the `configuration` section
   - This makes it more extensible because you're not tying the properties to what the customer has to provide since property values could potentially come from other sources.
   - There few cases where might want to have defined properties that the customer would not provide values for. For example:
     - cluster specific configuration like
       - Storage Class Name
       - Node name
     - Environment variables the device has available
       - identity provider information
       - admin user information
   - For Margo we could define a set of device specific parameters the device must be able to provide
     - May not need this now but we shouldn't prevent it from being possible in the future
     - We shouldn't prevent someone from doing it outside of the Margo spec (meaning it would only work for their app on their device and not officially supported)
   - For Margo we could make it possible for devices to have environment variables defined that could be used
     - If the device has the environment variable it would be used
     - If the device doesn't have the environment variable the customer would be prompted for the value.
     - As part of our device definition we could include the environment variables the device has (if any) that are useable for this purpose
     - May not need this now but we shouldn't prevent it from being possible in the future
     - We shouldn't prevent someone from doing it outside of the Margo spec (meaning it would only work for their app on their device and not officially supported)
10. The `sections` and `schema` are pretty much the same just under a `configuration` section instead of combined with the `properties`
