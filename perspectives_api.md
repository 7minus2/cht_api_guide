#Using the CloudHealth Perspective API
###Version 0.5 (2016-08-30)


##Overview
The CloudHealth Perspectives API allows for all CRUD operations on CloudHealth Perspectives.  A user can complete these CRUD operations via a Perspective schema JSON object. This schema can be retrieved by a READ operation for existing Perspectives.

Users can edit a schema file and upload it to an existing Perspective to modify rules, merges, perform re-ordering (among other things). Users can also post a schema to create a brand new Perspective, or delete a Perspective with these API calls.


##Schema fields
A Perspective schema has the following fields:

| **Schema Field**         | **Explanation**                                                                                                                                                                                                                                                                                                                |
|--------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ```name```               | The name of the Perspective. This does not have to match the name of any pre-existing (active or archived) Perspective.                                                                                                                                                                                                                       |
| ```include_in_reports``` | Whether the Perspective is included in Interactive Reports.                                                                                                                                                                                                                                                                    |
| ```rules```              | An array of rules (objects) that govern Perspective membership. The ordering of the rules matter; rules will be evaluated in order starting from the top. This part of the schema makes group reordering easy. Run a read operation to get the schema of a Perspective, change the order in the schema via your favorite editor and upload the modified schema to same Perspective. |
| ```merges```             | List of merge objects, which can be of type Dynamic Group or Dynamic Group Block. The Dynamic Group merge allows for two groups to merged and the Dynamic Block Group is to merge two entire blocks of group(s). Each merge specifies a list of source objects and a single target object (groups or blocks), where the sources are to be merged into the target.                                                                                             |
| ```constants```          | List of reference objects that refer to groups, blocks, and assets that are specified in rules. When a schema is retrieved through a read, every group (dynamic and static) is listed in the constants section. Although meant mainly for reference, constants can be modified to change the name of groups and blocks. The full list of potential constant types are listed at the bottom of the page. |

##Status Codes
Upon a successful call the API will return responses with corresponding http status:

| **Status Code** | **Meaning**           | **Comments**                                                                                                                                                                                             |
|-----------------|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 200             | OK                    | The request has succeeded.                                                                                                                                                                               |
| 400             | Bad Request           | This response means that server could not understand the request due to invalid syntax. The API will provide explanatory messages for bad requests _e.g. A Perspective schema requires a name declaration_ |
| 500             | Internal Server Error | The server has encountered a situation it doesn't know how to handle.                                                                                                                                    |


##OPERATIONS

Through all these operation described below you should be able to fully manage your Perspectives via the API. For all examples provided below, please note to replace:

  - `<api_key>` with your API Key
  - `<perspective_id>` with the required Perspective id

##Read Operation
A cURL invocation of the Read API call requires the id of the Perspective and the api key.
[See here on how to obtain an API key](https://docs.google.com/document/d/1eCNQwawgJVoYqCXoTgD95j_cvoqGNHMkchZDrcQHg54/edit#heading=h.1r6ts8di8kml).

###GET Action

```bash
GET "https://chapi.cloudhealthtech.com/v1/perspective_schemas/<perspective_id>?api_key=<api_key>"
```

####cURL Command to Get Perspective Schema

```bash
curl -s -H "Accept: application/json" "https://chapi.cloudhealthtech.com/v1/perspective_schemas/<perspective_id>?api_key=<api_key>"
```

**Or if you have python installed, you can format the response**

```bash
curl -s -H "Accept: application/json" "https://chapi.cloudhealthtech.com/v1/perspective_schemas/<perspective_id>?api_key=<api_key>" | python -m json.tool
```

#### Sample Response

```json
{
    "schema":
    {
        "constants": [
            {
                "list": [
                    {
                        "is_other": "true",
                        "name": "Other",
                        "ref_id": "206213093633"
                    }
                ],
                "type": "Group"
            }
        ],
        "include_in_reports": "true",
        "merges": [],
        "name": "API",
        "rules": []
    }
}
```



#### Sample Response - with Dynamic Group Block

```json
{
    "schema":
    {
        "constants": [
            {
                "list": [
                    {
                        "name": "Env",
                        "ref_id": "206159110488"
                    }
                ],
                "type": "Dynamic Group Block"
            },
            {
                "list": [
                    {
                        "blk_id": "206159110488",
                        "name": "production",
                        "ref_id": "206199274950",
                        "val": "production"
                    },
                    {
                        "blk_id": "206159110488",
                        "name": "feature",
                        "ref_id": "206199274960",
                        "val": "feature"
                    }
                ],
                "type": "Dynamic Group"
            },
            {
                "list": [
                    {
                        "is_other": "true",
                        "name": "Other",
                        "ref_id": "206195653674"
                    }
                ],
                "type": "Group"
            }
        ],
        "include_in_reports": "true",
        "merges": [],
        "name": "Environment ",
        "rules": [
            {
                "asset": "AwsAsset",
                "name": "Env",
                "ref_id": "206159110488",
                "tag_field": [
                    "cht_env"
                ],
                "type": "categorize"
            }
        ]
    }
}
```

##### Include Version for Perspective
`include_version`

If users want to retrieve the current version of the perspective, adding the `include_version=true` will return the current version of the perspective as an entry in the constants list.

Example:

```bash
curl -s -H "Accept: application/json" "https://chapi.cloudhealthtech.com/v1/perspective_schemas/<perspective_id>?api_key=...&include_version=true"
```


which returns an extra entry in the constants array:

```json
   { "type": "Version", "list": [ { "ref_id": 181, "val": 2 } ] }
```


##Index Operation

The index call will return the list of perspectives in the form of a hash. The hash contains the perspective id, perspective name, and an active flag field where a true implies active and false implies archived:

####cURL Command to Index Perspectives

```bash
curl -s -H "Accept: application/json" "https://chapi.cloudhealthtech.com/v1/perspective_schemas?api_key=<api key>"
```

####Sample Response

```json
{
  "206159639971": { "name": "DDE Test", "active": false  },
  "206159351171": { "name": "Environment ", "active": true  },
  "206159659286": { "name": "Environment-tmp", "active": false  },
  "206159657643": { "name": "Efe Test", "active": false  },
  "206159708697": { "name": "Sidd test", "active": true  },
  "181": { "name": "Function", "active": true  },
  "650": { "name": "Finance Costs", "active": true  }
}
```

By default the  will return all perspectives, active and archived. To only return active perspectives, add the argument `active_only=true`

####cURL Command to Index only Active Perspectives (non-archived)

```bash
curl -s -H "Accept: application/json" "https://chapi.cloudhealthtech.com/v1/perspective_schemas?api_key=<api key>&active_only=true"
```

#### Sample Response
```json
{
  "206159351171": { "name": "Environment ", "active": true  },
  "206159708697": { "name": "Fred test", "active": true  },
  "181": { "name": "Function", "active": true  },
  "650": { "name": "Finance Costs", "active": true  }
}
```


##Create Operation
To build a Perspective from scratch directly using the API use the POST action with the Perspective schema

###Create | Post Operation - cURL Command Format

```bash
curl -s -H "Content-Type: application/json" -XPOST "https://chapi.cloudhealthtech.com/v1/perspective_schemas/<perspective_id>?api_key=<api_key>" -d '{"schema": <schema JSON>}'
```

###Create | POST Operation - cURL Command Example

```bash
curl -H "Content-Type: application/json" -XPOST "https://chapi.cloudhealthtech.com/v1/perspective_schemas?api_key=<api_key>" -d '{"schema":{"name":"Environment-new","rules":[{"type":"categorize","asset":"AwsAsset","tag_field":["cht_env"],"ref_id":"206159110488","name":"Env"}],"merges":[],"constants":[{"ref_type":"Dynamic Group Block","ref_id":"206159110488","name":"Env"},{"ref_type":"Dynamic Group","ref_id":"206199274950","blk_id":"206159110488","val":"production","name":"production"},{"ref_type":"Dynamic Group","ref_id":"206199274960","blk_id":"206159110488","val":"feature","name":"feature"},{"ref_type":"Group","ref_id":"206195653674","name":"Other","is_other":"true"}]}}'
```

###Creating Duplicate Perspective
It is completely legal to extract a schema from a Perspective (e.g. Perspective A) and POST that schema after the name is changed (for example to B) to create a clone of A. All references in the schema rules to existing groups and blocks in (A) will just be seen as directive to create corresponding groups in B; it will not modify anything inside A.

###Creating New Group
Another way to put this is, if there is a reference to a nonexistent group (nonexistent inside the target Perspective) in a rule, the create/upload calls will create a brand new group inside the Perspective for it. This way, if you specify multiple rules referring to the same target string "my new group" inside the schema, the create/upload operations will create a new group and ensure these rules point to the newly created group.

For example, this following POST call will create a **new Perspective**:

```bash
curl -H "Content-Type: application/json" -XPOST "https://chapi.cloudhealthtech.com/v1/perspective_schemas?api_key=<api_key>" -d '{"schema":{"name":"Test 1000002","rules":[{"type":"filter","asset":"AwsInstance","to":"new group 1","condition":{"clauses":[{"field":["Active?"],"op":"=","val":"true"}]}},{"type":"filter","asset":"AwsInstance","to":"new group 1","condition":{"clauses":[{"field":["First Discovered"],"op":">","val":"2016-01-04T23:19:34+00:00"}]}}],"constants":[],"merges":[]}'
{"message":"Perspective 893353516727 created"}}
```

And when we do a Read call to fetch the schema for it after the Create operation, we'll see that;
a new group (Group-1) has been created and is displayed in the constants lists.
The references to "new group 1" will have been converted into references to the corresponding newly created group.

```bash
curl -H "Content-Type: application/json" -XPOST "https://chapi.cloudhealthtech.com/v1/perspective_schemas/<perspective_id>?api_key=<api_key>"{"name":"Test 1000004","include_in_reports":"false","rules":[{"type":"filter","asset":"AwsInstance","to":"893374827932","condition":{"clauses":[{"field":["Active?"],"op":"=","val":"true"}]}},{"type":"filter","asset":"AwsInstance","to":"893374827932","condition":{"clauses":[{"field":["First Discovered"],"op":"\u003E","val":"2016-01-04T23:19:34+00:00"}]}}],"merges":[],"constants":[{"ref_type":"Group","ref_id":"893374827931","name":"Other","is_other":"true"},{"ref_type":"Group","ref_id":"893374827932","name":"Group-1"}]}
```

##Update Operation

Update is similar to Create; in fact, Create essentially creates an empty Perspective and 'updates' that Perspective with the supplied schema. The actual Update call modifies the target Perspective to have rules as specified in the uploaded schema.

###Update | Put Operation - cURL Command Format

```bash
curl -s -H "Content-Type: application/json" -XPUT "https://chapi.cloudhealthtech.com/v1/perspective_schemas/<perspective_id>?api_key=<api_key>" -d '{"schema":<schema JSON>}'
```

**Note:**

  > This creates new groups when rules in the schema refer to groups that do not exist in the Perspective.

  > Any existing group that is not referred to by a rule inside the schema will be deleted, if the `allow_group_delete` option is set. By default this is unset, and the call will fail if all rules to an existing group deletion is required.

In addition to "to" fields in rules that specify target groups, you can specify a "from" group as well. This is typically not needed, as a missing "from" field, is interpreted as begin "from" the  Other (*Assets Not Allocated*) group.

When you create a group-to-group rule, the update/create calls verify that the source group already has at least one rule higher in the rule that targets it.

#### Avoiding conflicts for concurrent updates

#####check_version for Updates
If users want to enforce that they are not writing over an update made concurrently, they can send the version of the perspective (via a `check_version=<version>` argument) that they expect to be updating.

If the perspective has been updated (and therefore version incremented) since the last read operation, the update request will return a 400 error.

Example:

```bash
curl -s -H "Content-Type: application/json" -XPUT "https://chapi.cloudhealthtech.com/v1/perspective_schemas/<perspective_id>?api_key=<api_key>&check_version=3"
```

#####include_version
The version can be obtained by passing a `include_version=true` argument when making a read call to fetch a schema.



##Delete Operation

###Soft Delete

The following Delete call (the default option) soft deletes a Perspective if there are no dependences, such as, from policies or report subscriptions.

```bash
curl -s -H "Accept: application/json" -XDELETE "https://chapi.cloudhealthtech.com/v1/perspective_schemas/<perspective_id>?api_key=<api_key>"
```

###Force Delete
To delete the Perspective regardless of any dependency, you can add a **force** option, like so:

```bash
curl -s -H "Accept: application/json" -XDELETE "https://chapi.cloudhealthtech.com/v1/perspective_schemas/<perspective_id>?api_key=<api_key>&force=true"
```

###Hard Delete
Deletion, by default, is a soft delete. Deleted Perspective gets put into the Archive and can be resurrected (but any dependencies will have been dropped, they would have to be recreated). If we want to skip the archive and perform a hard-delete, there is the **hard_delete** option, like so:

```bash
curl -s -H "Accept: application/json" -XDELETE "https://chapi.cloudhealthtech.com/v1/perspective_schemas/<perspective id>?api_key=<api_key>&force=true&hard_delete=true"
```

### Allowed Constant Types

Constant Types can either be "Dynamic Group", "Dynamic Group", "Static Group", or an Asset Type listed below. (Note: Expansion Types are not included - e.g. AwsAsset or AzureTaggableAsset) 

* AwsAccount
* AwsAvailabilityZone
* AwsGroup
* AwsImage
* AwsInstance
* AwsInstanceReservation
* AwsInstanceStatus
* AwsInstanceType
* AwsRdsInstance
* AwsRegion
* AwsSecurityGroup
* AwsSecurityGroupRule
* AwsSnapshot
* AwsSpotRequest
* AwsTag
* AwsUser
* AwsVolume
* ChefAccount
* ChefCookbook
* ChefCookbookVersion
* ChefNode
* AwsS3Bucket
* AwsCloudFormationStack
* AwsCloudFormationResource
* ChefCookbookConstraint
* ChefEnvironment
* AwsRdsSnapshot
* AwsVpc
* AwsRedshiftCluster
* AwsRedshiftSnapshot
* AwsRedshiftReservedNode
* AwsVpcSubnet
* AwsLoadBalancer
* AwsDynamoDbTable
* AwsCacheCluster
* InstanceAttributeHash
* InstanceFileSystem
* PuppetAccount
* PuppetNode
* PuppetClass
* AwsRdsInstanceReservation
* AwsRdsSecurityGroup
* AwsRdsSecurityGroupRule
* AwsRdsSubnetGroup
* AwsInstanceReservationModification
* AwsInstanceReservationModificationItem
* AwsEmrCluster
* AwsEmrClusterInstanceGroup
* AwsEmrClusterInstance
* AwsInstanceReservationListing
* PagerdutyAccount
* PagerdutyIncident
* AwsNaclTable
* AwsNaclRule
* AwsCustomerGateway
* AwsVpnGateway
* AwsVpnConnection
* AwsInternetGateway
* AwsEni
* AwsElasticIp
* AwsEipPrivateIp
* Alert
* GcpComputeRegion
* GcpComputeZone
* GcpComputeProject
* GcpComputeMachineType
* GcpComputeInstance
* GcpComputeDiskType
* GcpComputeDisk
* GcpComputeSnapshot
* GcpComputeImage
* GcpComputeAttachedDisk
* GcpComputeMachineTypeProfile
* GcpTag
* GcpBillingStatement
* GcpBillingStatementItem
* GcpUsageStatement
* GcpStorageBucket
* VmwareAccount
* VmwareFolder
* VmwareDatacenter
* VmwareHost
* VmwareDatastore
* VmwareVirtualMachine
* VmwareCustomField
* DockerImage
* DockerContainer
* DelegatedAction
* CloudFrontDistribution
* CloudFrontOrigin
* DataCenterAccount
* DataCenterServer
* DataCenterServerFileSystem
* AzureEnrollment
* AzureBillingStatementItem
* AzureStoredBill
* AzureVm
* DataCenterServerNetworkInterface
* DataCenterServerCpu
* DataCenterServerBlockDevice
* AlertlogicAccount
* AwsRoute53HostedZone
* AnsibleAccount
* AnsibleNode
* AwsConfigRule
* AwsConfigRuleEvaluationResult
* AwsElasticacheReservedNode
* DatadogAccount
* DatadogHost
* DatadogTag
* AzureSubscription
* AzureTag
* AwsLambdaFunction
* AzureAccount
* AzureDepartment
* AzureCostCenter
* NewrelicAccount
* NewrelicServer
* IntegrationTag
* AzureZone
* AzureRegion
* AzureResourceGroup
* AzureServicePrincipal
* AzureVmFileSystem
* AzureVmNetworkInterface
* AzureVmCpu
* AzureVmVirtualDisk
* AzureVmSize
* AzureCatalog
* AzureVmRate
* AzureOffer
* AzureRateCard
* AzureStoredUsage
* AwsIamPolicy
* AwsIamRole
* AwsIamServerCertificate
* AwsAdsAgentStat
* AwsAdsServer
* AwsAccountRegionAttribute
* AwsAdsProcess
* AwsAdsConnection
* DataCenterTag
* CustomerTag
* AwsIamCredentialReport
* AzureCloudService
* AzureSecurityGroup
* AzureSecurityRule
* AzureIpAddress
* AwsIamPasswordPolicy
* AzureStorage
* AzureVmScaleSet
* AwsCloudtrailTrail
* AzureSqlServer
* AzureSqlDatabase
* AzurePartnerCenter
* AzureRedisCache
* JiraAccount
* IntegrationOauthAccount
* AzurePartnerCustomer
* IntegrationMetadata
* AzureVirtualNetworkGateway
* AzureAppServicePlan
* AzureServiceBusNamespace
* AzureVirtualNetwork
* AzureBatchAccount
* AzureBackupVault
* AzureRecoveryServicesVault
* AzureHdInsightCluster
* AzureSearchService
* AzureSqlDataWarehouse
* AzureStorSimpleDeviceManager
* AzureAppService
* AzureLogAnalyticsWorkspace
* AzureExpressRouteCircuit
* AzureCdnProfile
* AwsNatGateway
* AzureSnapshot
* AzureNetworkInterface
* AzureManagedDisk
* AzureAppInsight
* AwsElasticFileSystem
* AzureDocumentDb
* AzureKeyVault
* AwsElasticsearchDomain
* AwsElasticsearchInstance
* AwsElasticsearchVolume
* AwsDedicatedHost
* AzureApplicationGateway
* AwsConfigSetting
* AwsWorkspace
* AzureAppServiceEnvironment
* AwsAutoScalingGroup
* AwsKinesisStream
* AzureVmReservationRate
* AzureReservationOrder
* AzureReservation
* AzureVmRiQuote

