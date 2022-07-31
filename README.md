# delta-backend
This project deploys a complete backend for a [SpatioTemporal Asset Catalog](https://stacspec.org/) including a postgres database, a metadata API, and raster tiling API. Delta-backend is a non-forked version of the [eoAPI](https://github.com/developmentseed/eoAPI) demo project. Delta-backend is decoupled from the demo project to selectively incorporate new stable functionality from the fast moving development in eoAPI while providing a continuous baseline for delta-backend users and to support project specific business and deployment logic.

The primary tools employed in the [eoAPI demo](https://github.com/developmentseed/eoAPI) and this project are:
- [stac-spec](https://github.com/radiantearth/stac-spec)
- [stac-api-spec](https://github.com/radiantearth/stac-api-spec)
- [stac-fastapi](https://github.com/stac-utils/stac-fastapi)
- [pgstac](https://github.com/stac-utils/pgstac)
- [titiler](https://github.com/developmentseed/titiler)
- [titiler-pgstac](https://github.com/stac-utils/titiler-pgstac)

## Deployment
### Overview
TODO

### Other Documentation
#### Tooling
- [CDK Documentation] (https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html)

### Prerequisites
#### All Deployments
##### Tooling
-[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html)
-[AWS CDK](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_install)
#### Local Deployment
##### Tooling 
-[Docker](https://docs.docker.com/get-docker/)
#### Cloud deployment
##### Tooling 
TODO
##### Access
- Kion
- Stuff + Things?

### Enviroment variables

An [.example.env](.example.env) template is supplied for for local deployments. If updating an existing deployment, it is essential to check the most current values for these variables by fetching these values from AWS Secrets Manager. The environment secrets are named `delta-backend/<stage>-env`, for example the `dev` stack environment variables secret is named `delta-backend/dev-env`.

### Fetch environment variables using AWS CLI
To retrieve the variables for a stage that has been previously deployed, the secrets manager can be used to quickly populate an .env file. 
> Note: The environment variables stored as AWS secrets are manually maintained and should be reviewed before using.

```
export AWS_SECRET_ID=delta-backend/<stage>-env

aws secretsmanager get-secret-value --secret-id ${AWS_SECRET_ID} --query SecretString --output text | jq -r 'to_entries|map("\(.key)=\(.value|tostring)")|.[]' > .env
```

| Name | Explanation |
| --- | --- |
| `APP_NAME` | Optional app name used to name stack and resources, defaults to `delta-backend` |
| `STAGE` | **REQUIRED** Deployment stage used to name stack and resources, i.e. `dev`, `staging`, `prod` |
| `VPC_ID` | Optional resource identifier of VPC, if none a new VPC with public and private subnets will be provisioned. |
| `PERMISSIONS_BOUNDARY_POLICY` | Optional name of IAM policy to define stack permissions boundary |
| `CDK_DEFAULT_ACCOUNT` | When deploying from a local machine the AWS account id is required to deploy to an exiting VPC |
| `CDK_DEFAULT_REGION` | When deploying from a local machine the AWS region id is required to deploy to an exiting VPC |
| `DELTA_DB_PGSTAC_VERSION` | **REQUIRED** version of PgStac database, i.e. 0.5 |
| `DELTA_DB_SCHEMA_VERSION` | **REQUIRED** The version of the custom delta-backend schema, i.e. 0.1.1 |
| `DELTA_DB_SNAPSHOT_ID` | **Once used always REQUIRED** Optional RDS snapshot identifier to initialize RDS from a snapshot |
| `DELTA_DB_PRIVATE_SUBNETS` | Optional boolean to deploy database to private subnet |
| `DELTA_DOMAIN_HOSTED_ZONE_ID` | Optional Route53 zone identifier if using a custom domain name |
| `DELTA_DOMAIN_HOSTED_ZONE_NAME` | Optional custom domain name, i.e. delta-backend.xyz |
| `DELTA_DOMAIN_ALT_HOSTED_ZONE_ID` | Optional second Route53 zone identifier if using a custom domain name |
| `DELTA_DOMAIN_ALT_HOSTED_ZONE_NAME` | Optional second custom domain name, i.e. alt-delta-backend.xyz |
| `DELTA_DOMAIN_API_PREFIX` | Optional domain prefix override supports using a custom prefix instead of the STAGE variabe (an alternate version of the stack can be deployed with a unique STAGE=altprod and after testing prod API traffic can be cut over to the alternate version of the stack by setting the prefix to prod) |
| `DELTA_RASTER_ENABLE_MOSAIC_SEARCH` | Optional deploy the raster API with the mosaic/list endpoint TRUE/FALSE |
| `DELTA_RASTER_DATA_ACCESS_ROLE_ARN` | Optional arn of IAM Role to be assumed by raster-api for S3 bucket data access, if not provided default role for the lambda construct is used |

### CDK context
Currently a named version of the CDK toolkit is used for deployments. To use the default CDK toolkit bootstrapped for the account, remove `"@aws-cdk/core:bootstrapQualifier"` from the `"context"` in [`cdk.json`](cdk.json).

### Deploying the delta-backend

#### Local Docker Deployment
TODO

#### Cloud deployment

Install pre-requisites
```bash
nvm install 17
nvm use 17
node --version
npm install --location=global aws-cdk
python3 -m pip install --upgrade pip
python3 -m pip install -e ".[dev,deploy,test]"
```

Run the deployment
```
cdk synth
cdk deploy
```

## Deleting the stack

If this is a development stack that is safe to delete, you can delete the stack in Cloudformation console or via `cdk destroy`, however:

1. you will need to disable deletion protection and delete the database and delete the database
2. detach IGW from VPC and delete
3. delete the VPC (this should delete a subnet and security group too)

##### Checking status

After logging in to the console at https://<account number>.signin.aws.amazon.com/console the status of the CloudFormation stack can be viewed here: https://<aws-region>.console.aws.amazon.com/cloudformation/home.

The Cloudformation stack name is the combination of the app name and deployment stage environment variables https://github.com/NASA-IMPACT/delta-backend/blob/develop/config.py#L11

### Gotchas

#### General
TODO

### MCP requirements

In order to configure the lambda to be able to communicate with secretsmanager, a VPC interface endpoint must be configured.

```
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-XXX \
  --vpc-endpoint-type Interface \
  --service-name com.amazonaws.us-west-2.secretsmanager \
  --subnet-ids <private subnets of the lambda>
```

- added inbound rule of ALL type to security group attached to VPc endpoint (i think 443 may be ok)
- add configuration of endpoint_url as part of the boto3 client (not sure this is 100% necessary)


## Operations

## Ingesting metadata
STAC records should be loaded using [pypgstac](https://github.com/stac-utils/pgstac#pypgstac). The [cloud-optimized-data-pipelines](https://github.com/NASA-IMPACT/cloud-optimized-data-pipelines) project provides examples of cloud pipelines that use pypgstac to load data into a STAC catalog, as well as examples of transforming data to cloud optimized formats.

## Support scripts
Support scripts are provided for manual system operations.
- [Rotate pgstac password](support_scripts/README.md#rotate-pgstac-password)
## Usage examples: 

https://github.com/NASA-IMPACT/veda-documentation
## STAC community resources

### STAC browser
Radiant Earth's [stac-browser](https://github.com/radiantearth/stac-browser) is a browser for STAC catalogs. The demo version of this browser [radiantearth.github.io/stac-browser](https://radiantearth.github.io/stac-browser/#/) can be used to browse the contents of the delta-backend STAC catalog, paste the delta-backend stac-api URL deployed by this project in the demo and click load. Read more about the recent developments and usage of stac-browser [here](https://medium.com/radiant-earth-insights/the-exciting-future-of-the-stac-browser-2351143aa24b).

