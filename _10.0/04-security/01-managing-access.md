---
title: "Managing access"
excerpt: "Managing access for users and applications."
categories: security
slug: managing-access
toc: true
---

You can secure your {{site.data.reuse.long_name}} resources in a fine-grained manner by managing the access each user and application has to each resource.

## What resource types can I secure?

Within {{site.data.reuse.long_name}}, you can secure access to the following resource types, where the names in parentheses are the resource type names used in Access Control List (ACL) rules:
* Topics (`topic`): you can control the ability of users and applications to create, delete, read, and write to a topic.
* Consumer groups (`group`): you can control an application's ability to join a consumer group.
* Transactional IDs (`transactionalId`): you can control the ability to use the transaction capability in Kafka.

**Note:** Schemas in the {{site.data.reuse.short_name}} Schema Registry are a special case and are secured using the resource type of `topic` combined with a prefix of `__schema_`. You can control the ability of users and applications to create, delete, read, and update schemas.


## What roles can I assign?

Roles define the levels of access a user has to resources.

To provide applications access to Kafka resources, you create a `KafkaUser` custom resource.

To be able to create a `KafkaUser` custom resource in the {{site.data.reuse.short_name}} UI or CLI, you must have the Administrator role provided by the Identity and Access Management ([IAM](https://www.ibm.com/support/knowledgecenter/SSHKN6/iam/3.4.0/iam_guide.html){:target="_blank"}) service.


| Role             | Permitted actions | Example actions |
|:-----------------|:------------------|:----------------|
| Administrator    | Administrators have permissions to complete privileged actions. | Allow full access to all resources by assigning administrator access to the {{site.data.reuse.long_name}} instance. |


## Assigning access to users

If you have not set up {{site.data.reuse.icpcs}} teams, the default `admin` user has the `Cluster Administrator` role granting full access to all resources within the cluster. The generated password for this user is stored in a secret within the `ibm-common-services` namespace. To retrieve this password:

1. {{site.data.reuse.openshift_cli_login}}
2. Extract and decode the current {{site.data.reuse.icpcs}} admin password:

   `oc --namespace ibm-common-services get secret platform-auth-idp-credentials -o jsonpath='{.data.admin_password}' | base64 --decode`

**Note:** This password is auto-generated in accordance with the default password rules for {{site.data.reuse.icpcs}}. To change the username or password of the super user, see the instructions about [changing the cluster administrator access credentials](https://www.ibm.com/support/knowledgecenter/SSHKN6/iam/3.4.0/change_admin_passwd.html){:target="_blank"}.

The `admin` credentials can be used to [set up teams](https://www.ibm.com/support/knowledgecenter/SSHKN6/iam/3.4.0/teams.html){:target="_blank"} and then [add resources to the team](https://www.ibm.com/support/knowledgecenter/SSHKN6/iam/3.4.0/add_resource.html){:target="_blank"}. A team containing `Administrator` users that require access to the {{site.data.reuse.short_name}} instance must have a namespace resource added for the namespace that contains the {{site.data.reuse.short_name}} cluster.


## What are the available Kafka operations?

Access control in Apache Kafka is defined in terms of operations and resources. Operations are actions performed on a resource, and each operation maps to one or more APIs or requests.

| Resource type    | Operation        | Kafka API            |
|:-----------------|:-----------------|:---------------------|
| topic            | Alter            | CreatePartitions     |
|                  | AlterConfigs     | AlterConfigs         |
|                  | Create           | CreateTopics         |
|                  |                  | Metadata             |
|                  | Delete           | DeleteRecords        |
|                  |                  | DeleteTopics         |
|                  | Describe         | ListOffsets          |
|                  |                  | Metadata             |
|                  |                  | OffsetFetch          |
|                  |                  | OffsetForLeaderEpoch |
|                  | DescribeConfigs  | DescribeConfigs      |
|                  | Read             | Fetch                |
|                  |                  | OffsetCommit         |
|                  |                  | TxnOffsetCommit      |
|                  | Write            | AddPartitionsToTxn   |
|                  |                  | Produce              |
|                  | All              | All topic APIs       |
|     &nbsp;       |     &nbsp;       |        &nbsp;        |
| group            | Delete           | DeleteGroups         |
|                  | Describe         | DescribeGroup        |
|                  |                  | FindCoordinator      |
|                  |                  | ListGroups           |
|                  | Read             | AddOffsetsToTxn      |
|                  |                  | Heartbeat            |
|                  |                  | JoinGroup            |
|                  |                  | LeaveGroup           |
|                  |                  | OffsetCommit         |
|                  |                  | OffsetFetch          |
|                  |                  | SyncGroup            |
|                  |                  | TxnOffsetCommit      |
|                  | All              | All group APIs       |
|     &nbsp;       |     &nbsp;       |        &nbsp;        |
| transactionalId  | Describe         | FindCoordinator      |
|                  | Write            | AddOffsetsToTxn      |
|                  |                  | AddPartitionsToTxn   |
|                  |                  | EndTxn               |
|                  |                  | InitProducerId       |
|                  |                  | Produce              |
|                  |                  | TxnOffsetCommit      |
|                  | All              | All txnid APIs       |

### Implicitly-derived operations

Certain operations provide additional implicit operation access to applications.

When granted `Read`, `Write`, or `Delete`, applications implicitly derive the `Describe` operation.
When granted `AlterConfigs`, applications implicitly derive the `DescribeConfigs` operation.

For example, to `Produce` to a topic, the `Write` operation for the `topic` resource is required, which will implicitly derive the `Describe` operation required to get the topic metadata.


## Assigning access to applications

An {{site.data.reuse.short_name}} cluster can have up to `2` internal and `1` external Kafka listener configured. These listeners are the mechanisms by which Kafka client applications can communicate with the Kafka brokers. For the initial connection to the cluster, a bootstrap address is used which will resolve to one of the brokers in the cluster and respond with metadata describing all the relevant connection information for the remaining brokers.

Each Kafka listener exposing a connection to {{site.data.reuse.short_name}} can be secured with either Mutual TLS or SCRAM SHA 512 supported authentication mechanisms. The `KafkaUser` custom resource requires an authentication type defined in the `spec.authentication.type` field, with a value that matches the required Kafka listener authentication type. To assist in generating compatible `KafkaUser` credentials, the {{site.data.reuse.short_name}} UI indicates which authentication mechanism is being configured for each Kafka listener.

You can create a `KafkaUser` by using the {{site.data.reuse.short_name}} UI or CLI. It is also possible to create a `KafkaUser` by using the {{site.data.reuse.openshift_short}} UI or CLI, or the underlying Kubernetes API by applying a `KafkaUser` operand request.

### Access Control List (ACL) rules

Access to resources is assigned to applications through an Access Control List (ACL), which consists of rules. An ACL rule includes an operation on a resource together with the additional fields listed in the following tables. A `KafkaUser` custom resource represents the binding of an ACL to a principal, which is an entity that can be authenticated by the {{site.data.reuse.short_name}} instance.

An ACL rule adheres to the following schema:

| Property         | Type             | Description      |
|:-----------------|:-----------------|:-----------------|
| `host`           | string           | The host from which the action described in the ACL rule is allowed. |
| `operation`      | string           | The operation which will be allowed on the chosen resource. |
| `resource`       | object           | Indicates the resource for which the ACL rule applies. The `resource.type` value can be one of `topic`, `group`, or `transactionalId`. |

The resource objects used in ACL rules adhere to the following schema:

| Property         | Type             | Description      |
|:-----------------|:-----------------|:-----------------|
| `type`           | string           | Can be one of `topic`, `group`, or `transactionalId`. |
| `name`           | string           | Name of the resource for which the ACL rule applies. Can be combined with the `patternType` field to use the prefix pattern. |
| `patternType`    | string           | Describes the pattern used in the resource field. The supported types are `literal` and `prefix`. With literal pattern type, the resource field will be used as a definition of a full topic name. With prefix pattern type, the resource name will be used only as a prefix. <br> The default value is `literal`. |


Using the information about schemas and resource-operations described in the previous tables, the `spec.authorization.acls` list for a `KafkaUser` can be created as follows:
```
# ...
spec:
  # ...
  authorization:
    # ...
    acls:
      - host: '*'
        resource:
          type: topic
          name: 'client-'
          patternType: prefix
        operation: Write
```
In this example, an application using this `KafkaUser` would be allowed to write to any topic beginning with **client-** (for example, **client-records** or **client-billing**) from any host machine.

**Note:** The write operation also implicitly derives the required describe operation that Kafka clients require to understand the data model of a topic.

The following is an example ACL rule that provides access to read Schemas:
```
      - host: '*'
        resource:
          type: topic
          name: '__schema_'
          patternType: prefix
        operation: Read
```

Create a `KafkaUser` custom resource in the {{site.data.reuse.openshift_short}} web console by using the **IBM Event Streams** installed operator menu, select the **KafkaUser** tab to see a set of sample `KafkaUser` custom resource YAMLs to consume, produce, or modify every resource.
**Warning:** Do not use or modify the internal {{site.data.reuse.short_name}} `KafkaUser`s named `<cluster>-ibm-es-kafka-user` and `<cluster>-ibm-es-georep-source-user`. These are reserved to be used internally within the {{site.data.reuse.short_name}} instance.

### Creating a KafkaUser in the {{site.data.reuse.long_name}} UI

**Note:** If you have [configured the UI without security](../../installing/configuring/#configuring-ui-security) the **Connect to this cluster** section for generating credentials will be disabled. It is expected that all {{site.data.reuse.short_name}} components are also configured to be insecure when the UI is insecure which removes the need for credentials.

1. {{site.data.reuse.es_ui_login}}
2. Click the **Connect to this cluster** tile to view the **Cluster connection** panel.
3. Go to the **Kafka listener and Credentials** section.
4. To generate credentials for external clients, click **External**, or to generate credentials for internal clients, click **Internal**.
4. Click the **Generate SCRAM credential** or **Generate TLS credential** button next to the required listener to view the credential generation dialog.
5. Follow the instructions to generate the `KafkaUser` with the required permissions.

The generated credential appears after the listener bootstrap address:
* For SCRAM credentials, two tabs are displayed: **Username and password** and **Basic Authentication**. The SCRAM username and password combination is used by Kafka applications, while the Basic Authentication credential is for use as an HTTP authorization header.
* For TLS credentials, a download button is displayed, providing a zip archive with the required certificates and keys.

The cluster truststore is not part of the above credentials zip archive. This certificate is required for all external connections and is available to download from the **Cluster connection** panel under the **Certificates** heading. Upon downloading the PKCS12 certificate, the certificate password will also be displayed.

### Creating a KafkaUser in the {{site.data.reuse.long_name}} CLI

1. {{site.data.reuse.cp_cli_login}}
2. Initialize the {{site.data.reuse.short_name}} plugin specifying your namespace and choose your instance from the numbered list:\\
  `cloudctl es init -n <namespace>`
3. Use the `kafka-user-create` command to create a `KafkaUser` with the accompanying ACLs.
For example, to create SCRAM credentials with access to everything:\\
  ```
  cloudctl es kafka-user-create \
    --name my-user \
    --consumer \
    --producer \
    --schema-topic-create \
    --all-topics \
    --all-groups \
    --all-txnids \
    --auth-type scram-sha-512
  ```

For information about all options provided by the command, use the `--help` flag:\\
  `cloudctl es kafka-user-create --help`

The previous example creates a credential for all `topics`, `consumer-groups`, and `transactional-ids`. Additional options are also available to enable access to literals and prefixes for each resource.

### Retrieving credentials later

When a `KafkaUser` custom resource is created, the Entity Operator within {{site.data.reuse.short_name}} will create the principal in ZooKeeper with appropriate ACL entries. It will also create a Kubernetes `Secret` that contains the Base64-encoded SCRAM password for the `scram-sha-512` authentication type, or the Base64-encoded certificates and keys for the `tls` authentication type.

You can retrieve the credentials later in the {{site.data.reuse.openshift_short}} by using the name of the KafkaUser. For example, to retrieve the credentials by using the {{site.data.reuse.openshift_short}} CLI:

1. {{site.data.reuse.openshift_cli_login}}
2. Use the following command retrieve the reqiured `KafkaUser` data, adding the `KafkaUser` name and your chosen namespace:\\
  \\
  `oc get kafkauser <name> --namespace <namespace> -o jsonpath='{"username: "}{.status.username}{"\nsecret-name: "}{.status.secret}'`\\
  \\
  The command results in two lines of output, containing:
  - the principal username
  - the namespaced-name of the Kubernetes `Secret` containing the SCRAM password or the TLS certificates.
3. Decode the credentials.\\
   \\
   For SCRAM, use the `secret-name` from step 2 to get the password and decode it:\\
  \\
  `oc get secret <secret-name> --namespace <namespace> -o jsonpath='{.data.password}' | base64 --decode`\\
  \\
  \\
  For TLS, get the credentials, decode them, and write each certificates and keys to files:\\
  \\
  `oc get secret <secret-name> --namespace <namespace> -o jsonpath='{.data.ca\.crt}' | base64 --decode > ca.crt`\\
  \\
  `oc get secret <secret-name> --namespace <namespace> -o jsonpath='{.data.user\.crt}' | base64 --decode > user.crt`\\
  \\
  `oc get secret <secret-name> --namespace <namespace> -o jsonpath='{.data.user\.key}' | base64 --decode > user.key`\\
  \\
  `oc get secret <secret-name> --namespace <namespace> -o jsonpath='{.data.user\.p12}' | base64 --decode > user.p12`\\
  \\
  `oc get secret <secret-name> --namespace <namespace> -o jsonpath='{.data.user\.password}' | base64 --decode`

The cluster truststore certificate is required for all external connections and is available to download from the **Cluster connection** panel under the **Certificates** heading. Upon downloading the PKCS12 certificate, the certificate password will also be displayed.

Similarly, these `KafkaUser` and `Secret` resources can be inspected by using the {{site.data.reuse.openshift_short}} web console.

**Warning:** Do not use or modify the internal {{site.data.reuse.short_name}} `KafkaUser`s named `<cluster>-ibm-es-kafka-user` and `<cluster>-ibm-es-georep-source-user`. These are reserved to be used internally within the {{site.data.reuse.short_name}} instance.


## Revoking access for an application

As each application will be using credentials provided through a `KafkaUser` instance, deleting the `KafkaUser` custom resource will revoke access for that application. Similarly, removing ACL rules from a `KafkaUser` custom resource will remove the associated resource operations that were being permitted by the rule.
