Deploy CRD and operator:

```
kubectl create ns operators
kubectl -n operators apply -f deploy/mandatory.yaml
```

Create cluster:

```
kubectl create ns confluent
kubectl -n confluent apply -f deploy/confluent-platform.yaml
```


```
---
apiVersion: pm.bet/v1alpha1
kind: ConfluentPlatform
metadata:
  name: confluent
spec:
  # Version of Confluent Platform.
  version: 5.5.0

  license: ""

  zookeeper:
    enabled: true

    # Number of nodes of the zookeeper cluster. The amount must be odd.
    size: 3

    # This is the port where ZooKeeper clients will listen on.
    # This is where the Brokers will connect to ZooKeeper.
    # Typically this is set to 2181.
    clientPort: 2181

    # This port is used by followers to connect to the active leader.
    # This port should be open between all ZooKeeper ensemble members.
    serverPort: 2888

    # This port is used to perform leader elections between ensemble members.
    # This port should be open between all ZooKeeper ensemble members.
    electionPort: 3888

    # JMX port
    jmxPort: 9999

    # The unit of time for ZooKeeper translated to milliseconds.
    # This governs all ZooKeeper time dependent operations.
    # It is used for heartbeats and timeouts especially.
    # Note that the minimum session timeout will be two ticks.
    tickTime: 2000

    # The initLimit and syncLimit are used to govern how long following ZooKeeper servers can take
    # to initialize with the current leader and how long they can be out of sync with the leader.
    initLimit: 5
    syncLimit: 2

    # When enabled, ZooKeeper auto purge feature retains the autopurge.snapRetainCount most recent snapshots
    # and the corresponding transaction logs in the dataDir and dataLogDir respectively and deletes the rest.
    autopurgeSnapRetainCount: 3

    # The time interval in hours for which the purge task has to be triggered.
    # Set to a positive integer (1 and above) to enable the auto purging.
    autopurgePurgeInterval: 24

    # The maximum allowed number of client connections for a ZooKeeper server.
    # To avoid running out of allowed connections set this to 0 (unlimited).
    maxClientCnxns: 60

    logRootLoglevel: "INFO"
    logLoggers: "INFO"

    persistent: true
    dataDirSize: 1Gi
    datalogDirSize: 1Gi

  kafka:
    enabled: true
    persistent: true
    storageSize: 5Gi
    size: 3
    plaintextPort: 9092
    sslPort: 9093
    saslPort: 9094
    jmxPort: 9999
    allowEveryoneIfNoAclFound: true
    alterLogDirsReplicationQuotaWindowNum: 11
    alterLogDirsReplicationQuotaWindowSizeSeconds: 1
    authorizerClassName: kafka.security.auth.SimpleAclAuthorizer
    autoCreateTopicsEnable: true
    autoLeaderRebalanceEnable: true
    backgroundThreads: 10
    brokerIdGenerationEnable: true
    brokerRack: kubernetes
    comperssionType: producer
    connectionsMaxIdleMs: 600000
    connectionFailedAuthenticationDelayMs: 100
    controlledShutdownEnable: true
    controlledShutdownMaxRetries: 3
    controlledShutdownRetryBackoffMs: 5000
    controlledSocketTimeoutMs: 30000
    defaultReplicationFactor: 3
    delegationTokenExpiryCheckIntervalMs: 3600000
    delegationTokenExpiryTimeMs: 86400000
    delegationTokenMaxLifetimeMs: 604800000
    deleteRecordPurgatoryPurgeIntervalRequests: 1
    deleteTopicEnable: true
    fetchPurgatoryPurgeIntervalRequests: 1000
    groupInitialRebalanceDelayMs: 3000
    groupMaxSessionTimeoutMs: 300000
    groupMinSessionTimeoutMs: 6000
    leaderImbalanceCheckIntervalSeconds: 300
    leaderImbalancePerBrokerPercentage: 10
    log4jRootLoglevel: INFO
    logCleanerBackoffMs: 15000
    logCleanerDedupeBufferSize: 134217728
    logCleanerDeleteRetentionMs: 86400000
    logCleanerEnable: true
    logCleanerIoBufferLoadFactor: 0.9
    logCleanerIoBufferSize: 524288
    logCleanerIoMaxBytesPerSecond: "1.7976931348623157E308"
    logCleanerMinCleanableRatio: 0.5
    logCleanerMinCompactionLagMs: 0
    logMessageTimestampType: CreateTime

  schemaRegistry:
    enabled: true
    size: 1
    port: 8081
    jmxPort: 9999
    avro_compatibility_level: backward
    compression_enable: true

  kafkaConnect:
    enabled: true
    size: 1
    port: 8083
    jmxPort: 9999
    configStorageTopic: connect-configs
    groupId: connect
    keyConverter: org.apache.kafka.connect.json.JsonConverter
    offsetStorageTopic: connect-offsets
    statusStorageTopic: connect-status
    valueConverter: org.apache.kafka.connect.json.JsonConverter

  ksqldbServer:
    enabled: true
    size: 1
    port: 8088
    jmxPort: 9999
    ksqlAccessValidatorEnable: autooo

  kafkaRest:
    enabled: true
    size: 1
    port: 8082
    jmxPort: 9999
    compressionEnable: true

  controlCenter:
    enabled: true
    size: 1
    port: 9021
    jmxPort: 9999

    # Auto create a trigger and an email action for Control Center’s cluster down alerts.
    alertClusterDownAutocreate: false

    # Send rate per hour for auto-created cluster down alerts.
    # Default: 12 times per hour (every 5 minutes).
    alertClusterDownSendRate: 12

    # Email to send alerts to when Control Center’s cluster is down.
    # alertClusterDownToEmail: duty@example.com

    # The PagerDuty integration key to post alerts to a certain service when Control Center’s cluster is down.
    # alertClusterDownToPagerdutyIntegrationkey: xxxxxx

    # The Slack webhook URL to post alerts to when Control Center’s cluster is down.
    # alertClusterDownToWebhookurlSlack: xxxxxx

    # The maximum number of trigger events in one alert.
    alertMaxTriggerEvents: 1000

    # JWT token issuer.
    authBearerIssuer: Confluent

    # List of roles with limited access. No editing or creating using the UI.
    # Any role here must also be added to confluent.controlcenter.rest.authentication.roles
    # authRestrictedRoles: []

    # Timeout in milliseconds after which a user session will have to be re-authenticated
    # with the authentication service (e.g. LDAP). Defaults to 0, which means authentication is done for every request.
    # Increase this value to avoid calling the LDAP service for each request.
    authSessionExpirationMs: 0

    # Enable user access to Edit dynamic broker configuration settings.
    brokerConfigEditEnable: false

    # Command streams start timeout
    commandStreamsStartTimeout: 300000

    # Topic used to store Control Center configuration.
    commandTopic: _confluent-command

    # Retention for command topic.
    commandTopicRetentionMs: 259200000

    # Time to wait when attempting to retrieve Consumer Group metadata.
    consumerMetadataTimeoutMs: 15000

    # Enable the Consumers view in Control Center.
    consumersViewEnable: true

    # Enable deprecated Streams Monitoring and System Health views.
    deprecatedViewsEnable: false

    # Threshold for the max difference in disk usage across all brokers before disk skew warning is published.
    diskSkewWarningMinBytes: 1073741824

    
    internalStreamsStartTimeout: 21600000
    internalTopicsChangelogSegmentBytes: 134217728
    internalTopicsPartition: 4
    internalTopicsReplication: 3
    internalTopicsRetentionBytes: -1
    internalTopicsRetentionMs: 604800000

    # Enable user access to the ksqlDB GUI.
    ksqlEnable: true

    # License Manager topic
    licenseManager: _confluent-controlcenter-license-manager-5-5-0

    # Enable License Manager in Control Center.
    licenseManagerEnable: true

    # Override for mailFrom config to send message bounce notifications.
    # mailBounceAddress: 

    # Enable email alerts. If this setting is false, you cannot add email alert actions in the web user interface.
    mailEnabled: false

    # The originating address for emails sent from Control Center.
    mailFrom: c3@confluent.io

    # Hostname of outgoing SMTP server.
    mailHostName: localhost

    # Password for username/password authentication.
    # mailPassword: 

    # SMTP port open on mailHostName.
    mailPort: 587

    # SMTP port open on confluent.controlcenter.mail.host.name.
    mailSslCheckServerIdentity: false

    # Forces using STARTTLS.
    mailStarttlsRequired: false

    # Username for username/password authentication.
    # Authentication with your SMTP server only performs if this value is set.
    # mailUsername: 

    # Control Center Name.
    name: _confluent-controlcenter

    productiveSupportUiCtaEnable: false
    restCompressionEnable: true
    restHstsEnable: true

    # REST port.
    restPort: 9021

    sbkUiEnable: false

    # Enable user access to Manage Schemas for Topics.
    schemaRegistryEnable: true

    # The interval (in seconds) used for checking the health of Confluent Platform nodes.
    # This includes ksqlDB, Connect, Schema Registry, REST Proxy, and Metadata Service (MDS).
    serviceHealthcheckIntervalSec: 20

    # Maximum number of memory bytes used for record caches across all threads.
    streamsCacheMaxBytesBuffering: 1073741824

    streamsConsumerSessionTimeoutMs: 60000
    streamsNumStreamThreads: 8

    # Compression type to use on internal topic production.
    streamsProducerCompressionType: lz4

    streamsProducerDeliveryTimeoutMs: 2147483647
    streamsProducerLingerMs: 500
    streamsProducerMaxBlockMs: "9223372036854775807"
    streamsProducerRetries: 2147483647
    streamsProducerRetryBackofMs: 100

    # Number of times to retry client requests failing with transient errors.
    # Does not apply to producer retries, which are defined using the streamsProducerRetries setting described below.
    streamsRetries: 2147483647

    streams_upgrade_from: 2.3

    # Enable users to inspect topics.
    topicInspectionEnable: true

    triggerActiveControllerCountEnable: false

    # Enable auto updating the Control Center UI.
    uiAutoupdateEnable: true

    # Enable the Active Controller chart to display within the Broker uptime panel in the Control Center UI.
    uiControllerChartEnable: true

    # Configure a threshold (in seconds) before data is considered out of date. Default: 120 seconds (2 minutes).
    uiDataExpiredThreshold: 120

    # Enable Replicator monitoring in the Control Center UI.
    uiReplicatorMonitoringEnable: true

    # Enable or disable usage data collection in Control Center.
    usageDataCollectionEnable: true

    # Enable supported webhook alerts. If this setting is false, you cannot add webhook alert actions in the web user interface.
    webhookEnabled: true

```