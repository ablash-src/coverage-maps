# ECS Coverage Analysis Deployment Guide

## Overview
This guide documents the complete ECS deployment process for the Coverage Analysis Python script and future maintenance procedures.

**Deployment Date**: August 20, 2026  
**Architecture**: EventBridge → ECS Fargate → S3  
**Schedule**: Every Monday at 2 AM PST (`cron(0 10 ? * MON *)`)

## Initial Deployment (One-Time Setup) ✅ COMPLETED

### What Was Deployed
- **ECS Cluster**: `coverage-analysis-cluster`
- **Task Definition**: `coverage-analysis:2` (4 vCPU, 16GB RAM)
- **EventBridge Rule**: Weekly Monday 2 AM PST trigger
- **ECR Repository**: `coverage-analysis`
- **Docker Image**: Python 3.11 with pandas, numpy, boto3
- **IAM Roles**: S3 access to `wwrr-return-data` bucket

### Files Created
```
coverage-analysis/
├── us_live_coverage_analysis.py    # Main Python script
├── Dockerfile                      # Container definition
├── requirements.txt                # Python dependencies
├── cloudformation-template.yaml    # ECS infrastructure
├── deploy.sh                       # Deployment script
└── ECS-Coverage-Analysis-Guide.md  # This documentation
```

### Initial Deployment Steps (COMPLETED)
1. Created all 5 files
2. Uploaded to AWS CloudShell
3. Fixed Dockerfile typo (`apt-get update`)
4. Fixed CloudFormation subnets configuration
5. Deleted failed stack and redeployed
6. ✅ **Successfully deployed and running**

---

## Future Updates: Python Script Changes

### When You Need to Update us_live_coverage_analysis.py

#### Option 1: Quick Update (Recommended)
```bash
# 1. Update us_live_coverage_analysis.py locally
# 2. Upload only the updated .py file to CloudShell  
# 3. Run the deployment script
./deploy.sh
```

The script is smart enough to:
- ✅ Skip CloudFormation deployment (no changes needed)
- ✅ Rebuild Docker image with new Python code
- ✅ Push updated image to ECR
- ✅ ECS will use new image on next run

#### Option 2: Manual Docker Commands
```bash
# Build new image
docker build -t coverage-analysis .

# Tag for ECR
docker tag coverage-analysis:latest 883765745396.dkr.ecr.us-west-2.amazonaws.com/coverage-analysis:latest

# Push to ECR
docker push 883765745396.dkr.ecr.us-west-2.amazonaws.com/coverage-analysis:latest
```

### Update Process Summary
1. **Edit** `us_live_coverage_analysis.py` locally
2. **Upload** updated file to CloudShell
3. **Run** `./deploy.sh` (takes ~2-3 minutes)
4. **Done!** Next Monday 2 AM PST will use new code

---

## When Full Redeployment Is Needed

### Infrastructure Changes Requiring Full Deploy:
- **Schedule changes** (different time/frequency)
- **Resource changes** (CPU/RAM adjustments)
- **New Python dependencies** (updating requirements.txt)
- **Container changes** (Dockerfile modifications)
- **IAM permissions** (new S3 buckets, etc.)

### Full Redeployment Process:
1. Update the relevant files (CloudFormation, Dockerfile, etc.)
2. Upload all changed files to CloudShell
3. Run `./deploy.sh`
4. May need to delete existing stack if major changes

---

## Monitoring and Troubleshooting

### Check Task Execution
```bash
# List recent tasks
aws ecs list-tasks --cluster coverage-analysis-cluster

# Get task details
aws ecs describe-tasks --cluster coverage-analysis-cluster --tasks TASK_ARN
```

### View Logs
```bash
# CloudWatch logs
aws logs get-log-events \
  --log-group-name "/ecs/coverage-analysis" \
  --log-stream-name "coverage-analysis-container/coverage-analysis/TASK_ID"
```

### Manual Test Run
```bash
# Trigger manually for testing
aws ecs run-task \
  --cluster coverage-analysis-cluster \
  --task-definition coverage-analysis \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={assignPublicIp=ENABLED}"
```

### Common Issues

#### Docker Build Fails
- Check Dockerfile syntax
- Verify all files are uploaded to CloudShell
- Run `docker build -t coverage-analysis . --no-cache`

#### Task Fails to Start
- Check IAM role permissions
- Verify ECR image exists: `aws ecr describe-images --repository-name coverage-analysis`
- Check CloudWatch logs for startup errors

#### S3 Access Issues
- Ensure `wwrr-return-data` bucket permissions
- Verify IAM role has S3 access
- Check bucket name in Python script

---

## Current Configuration

### Deployment Details
- **AWS Account**: 883765745396
- **Region**: us-west-2
- **ECR Repository**: `883765745396.dkr.ecr.us-west-2.amazonaws.com/coverage-analysis`
- **S3 Bucket**: `wwrr-return-data`

### Resource Allocation
- **CPU**: 4 vCPU (4096 CPU units)
- **Memory**: 16GB (16384 MB)
- **Timeout**: Unlimited (no Lambda 15-minute limit)
- **Network**: Public IP enabled for ECR/S3 access

### Schedule
- **Expression**: `cron(0 10 ? * MON *)`
- **Description**: Monday 2 AM PST (10 AM UTC)
- **Timezone**: PST (Pacific Standard Time)

### Output Location
- **S3 Path**: `s3://wwrr-return-data/wwrr-coverage/US/Output Files/`
- **Files Generated**:
  - `Demand Coverage US Monthly.csv000`
  - `Density Coverage US Monthly.csv000`
  - `US Returns Store Mapping.csv000`
  - `US_Returns_Store_Mapping.geojson`

---

## Architecture Benefits

### ECS vs Lambda Comparison
| Feature | Lambda (Old) | ECS Fargate (New) |
|---------|-------------|-------------------|
| **Runtime Limit** | 15 minutes | Unlimited |
| **Memory** | 10GB max | 16GB+ available |
| **CPU** | Limited | 4 vCPU dedicated |
| **Cold Starts** | Yes | No |
| **Cost** | Pay per execution | Pay for actual usage |
| **Complexity** | Packaging issues | Simple Docker |

### Why Direct EventBridge → ECS?
- ✅ **No Lambda wrapper needed**
- ✅ **Fewer moving parts**
- ✅ **Better performance**
- ✅ **Simpler debugging**
- ✅ **No packaging size limits**

---

## Cost Analysis

### Weekly Run (Monday 2 AM PST)
- **Estimated Runtime**: ~2 hours
- **ECS Fargate Cost**:
  - 4 vCPU: ~$0.20/hour × 2 hours = $0.40
  - 16GB RAM: ~$0.09/hour × 2 hours = $0.18
  - **Total per run**: ~$0.58

### Monthly Cost
- **4 runs per month**: ~$2.32
- **CloudWatch logs**: ~$0.50
- **ECR storage**: ~$0.10
- **Total monthly**: ~$2.92

**Much cheaper than keeping a large Lambda function running!**

---

## Quick Reference Commands

### Update Python Script
```bash
# 1. Upload new us_live_coverage_analysis.py to CloudShell
# 2. Run deployment
./deploy.sh
```

### Check Current Status
```bash
# View scheduled rule
aws events describe-rule --name coverage-analysis-ecs-CoverageAnalysisSchedule-*

# Check ECS cluster
aws ecs describe-clusters --clusters coverage-analysis-cluster

# View task definition
aws ecs describe-task-definition --task-definition coverage-analysis
```

### Emergency Commands
```bash
# Disable schedule (stop auto-runs)
aws events disable-rule --name coverage-analysis-ecs-CoverageAnalysisSchedule-*

# Enable schedule (resume auto-runs)  
aws events enable-rule --name coverage-analysis-ecs-CoverageAnalysisSchedule-*

# Delete entire stack (nuclear option)
aws cloudformation delete-stack --stack-name coverage-analysis-ecs
```

---

## Support Information

### CloudFormation Stack
- **Name**: `coverage-analysis-ecs`
- **Status**: CREATE_COMPLETE
- **Template**: `cloudformation-template.yaml`

### Key ARNs
- **ECS Cluster**: `arn:aws:ecs:us-west-2:883765745396:cluster/coverage-analysis-cluster`
- **Task Definition**: `arn:aws:ecs:us-west-2:883765745396:task-definition/coverage-analysis:2`
- **EventBridge Rule**: `arn:aws:events:us-west-2:883765745396:rule/coverage-analysis-ecs-CoverageAnalysisSchedule-*`

### Deployment History
- **August 20, 2026**: Initial ECS deployment successful
- **Previous**: Lambda-based deployment (deprecated)

---

## Manual Task Execution

### When You Need Manual Runs
Sometimes you need to run the coverage analysis outside the scheduled Monday 2 AM PST runs:
- Testing after code changes
- Processing urgent data updates
- Ad-hoc analysis requests
- Troubleshooting deployment issues

### Method 1: AWS CLI (CloudShell) - Quick & Scriptable

```bash
# Basic manual run
aws ecs run-task \
  --cluster coverage-analysis-cluster \
  --task-definition coverage-analysis \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={assignPublicIp=ENABLED}"
```

#### Create a Reusable Script:
```bash
# Create manual-run.sh for easy execution
cat > manual-run.sh << 'EOF'
#!/bin/bash
echo "🚀 Starting manual coverage analysis run..."

TASK_ARN=$(aws ecs run-task \
  --cluster coverage-analysis-cluster \
  --task-definition coverage-analysis \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={assignPublicIp=ENABLED}" \
  --query 'tasks[0].taskArn' \
  --output text)

echo "✅ Task started: $TASK_ARN"
echo "📊 Monitor progress in ECS Console or CloudWatch Logs"
echo "📁 Output will be in: s3://wwrr-return-data/wwrr-coverage/US/Output Files/"
EOF

chmod +x manual-run.sh
```

**Usage**: `./manual-run.sh`

### Method 2: AWS Console (GUI) - Visual & User-Friendly

1. **Navigate to ECS**:
   - Go to **ECS Console** → **Clusters** → `coverage-analysis-cluster`

2. **Start New Task**:
   - Click **"Tasks"** tab → **"Run new Task"**

3. **Configure Task**:
   - **Launch type**: `Fargate`
   - **Task Definition**: `coverage-analysis` (should auto-select latest revision)
   - **Platform version**: `LATEST`
   - **Cluster**: `coverage-analysis-cluster` (pre-filled)

4. **Networking** (auto-configured):
   - **VPC**: Uses default VPC
   - **Subnets**: Auto-selected public subnets
   - **Security Groups**: Default security group
   - **Auto-assign public IP**: `ENABLED`

5. **Execute**:
   - Click **"Run Task"**
   - Task will appear in Tasks list with status **Pending** → **Running**

### Method 3: EventBridge Manual Trigger (Advanced)

```bash
# Trigger the EventBridge rule manually (simulates the schedule)
aws events put-events \
  --entries Source=manual,DetailType=test,Detail='{}'
```

**Note**: This method simulates the actual schedule trigger but may not work exactly the same as direct ECS task execution.

### Monitoring Manual Runs

#### Check Task Status:
```bash
# List currently running tasks
aws ecs list-tasks --cluster coverage-analysis-cluster --desired-status RUNNING

# Get detailed task information
aws ecs describe-tasks --cluster coverage-analysis-cluster --tasks TASK_ARN
```

#### View Real-Time Logs:
1. **CloudWatch Console**:
   - Go to **CloudWatch** → **Log groups** → `/ecs/coverage-analysis`
   - Click **"Log streams"** tab
   - Select the most recent stream (newest timestamp)
   - Click **"Resume"** to see real-time logs

2. **CloudShell/CLI**:
```bash
# View live logs
aws logs tail /ecs/coverage-analysis --follow --since 1m
```

#### Expected Task Lifecycle:
1. **Created** (0-5 seconds): Task definition instantiated
2. **Provisioning** (5-15 seconds): Resources allocated  
3. **Pending** (15-60 seconds): Docker image download
4. **Pull started/completed** (30-90 seconds): Container setup
5. **Running** (2+ hours): Python script execution
6. **Stopped** (exit code 0 = success): Task completed

### Manual Run Behavior

#### Expected Performance:
- **Duration**: ~2 hours (same as scheduled runs)
- **Resources**: 4 vCPU, 16GB RAM
- **Cost**: ~$0.58 per manual run
- **Output Location**: Same S3 path as scheduled runs

#### Output Files Generated:
- `s3://wwrr-return-data/wwrr-coverage/US/Output Files/Demand Coverage US Monthly.csv000`
- `s3://wwrr-return-data/wwrr-coverage/US/Output Files/Density Coverage US Monthly.csv000`
- `s3://wwrr-return-data/wwrr-coverage/US/Output Files/US Returns Store Mapping.csv000`
- `s3://wwrr-return-data/wwrr-coverage/US/Output Files/Geodata/US_Returns_Store_Mapping.geojson`

#### Verify Success:
```bash
# Check if output files were created
aws s3 ls s3://wwrr-return-data/wwrr-coverage/US/Output\ Files/ --recursive --human-readable

# Check task exit status (0 = success)
aws ecs describe-tasks --cluster coverage-analysis-cluster --tasks TASK_ARN \
  --query 'tasks[0].containers[0].exitCode'
```

### Manual Run vs Scheduled Run

| Aspect | Manual Run | Scheduled Run |
|--------|------------|---------------|
| **Trigger** | On-demand | Monday 2 AM PST |
| **Duration** | ~2 hours | ~2 hours |
| **Resources** | 4 vCPU, 16GB RAM | 4 vCPU, 16GB RAM |
| **Output** | Same S3 location | Same S3 location |
| **Cost** | ~$0.58 per run | ~$0.58 per run |
| **Monitoring** | Manual checking | Automated alerts |

### Troubleshooting Manual Runs

#### Task Fails to Start:
- Check ECS service limits in your region
- Verify ECR repository contains latest image
- Ensure VPC has available capacity

#### Task Stops Immediately:
- Check CloudWatch logs for Python errors
- Verify IAM role has S3 permissions
- Confirm Docker image is valid

#### No Output Files:
- Check S3 bucket permissions
- Verify input data exists in S3
- Review CloudWatch logs for processing errors

---

## Lambda Map Generator

### Overview
An automated Lambda function that generates interactive HTML maps whenever GeoJSON files are created by the ECS coverage analysis pipeline.

**Architecture**: ECS → S3 GeoJSON → S3 Event → Lambda → HTML Map → S3

### Initial Setup (One-Time)
1. Upload Lambda files to CloudShell:
   - `lambda_map_generator.py`
   - `lambda-map-generator-stack.yaml`
   - `deploy-map-generator.sh`

2. Deploy Lambda function:
```bash
chmod +x deploy-map-generator.sh
./deploy-map-generator.sh
```

3. Configure S3 trigger manually in AWS Console:
   - **Bucket**: wwrr-return-data
   - **Event**: s3:ObjectCreated:*
   - **Prefix**: wwrr-coverage/
   - **Suffix**: .geojson
   - **Destination**: geojson-map-generator Lambda function

### Lambda Code Updates

When you need to modify `lambda_map_generator.py`:

#### Development Workflow:
1. **Edit** `lambda_map_generator.py` locally
2. **Upload** to CloudShell  
3. **Update**: `./update-lambda.sh`
4. **Test**: `./manage-lambda.sh test`
5. **Verify**: Check S3 for generated HTML map
6. **Debug**: `./manage-lambda.sh logs` if issues

#### Required Scripts:
Create these helper scripts in CloudShell:

```bash
# Create update script
cat > update-lambda.sh << 'EOF'
#!/bin/bash
echo "🔄 Updating Lambda function..."
zip -j lambda-deployment.zip lambda_map_generator.py
aws lambda update-function-code \
    --function-name geojson-map-generator \
    --zip-file fileb://lambda-deployment.zip \
    --region us-west-2
rm lambda-deployment.zip
echo "✅ Lambda updated!"
EOF

# Create management script
cat > manage-lambda.sh << 'EOF'
#!/bin/bash
function test_lambda() {
    aws lambda invoke \
        --function-name geojson-map-generator \
        --payload '{
            "Records": [{
                "s3": {
                    "bucket": {"name": "wwrr-return-data"},
                    "object": {"key": "wwrr-coverage/US/Output Files/Geodata/US_Returns_Store_Mapping.geojson"}
                }
            }]
        }' test-response.json
    cat test-response.json && rm test-response.json
}

function check_logs() {
    aws logs tail /aws/lambda/geojson-map-generator --since 10m
}

case $1 in
    "test") test_lambda ;;
    "logs") check_logs ;;
    *) echo "Usage: $0 {test|logs}" ;;
esac
EOF

chmod +x update-lambda.sh manage-lambda.sh
```

### Manual Lambda Testing & Triggers

#### Option 1: Lambda Console Test (Recommended for Testing)

🖥️ **Lambda Console Test Steps**:

1. **Go to Lambda Console**
   - AWS Console → Lambda → Functions → geojson-map-generator

2. **Create Test Event**
   - Click "Test" button (top right)
   - Create new event → Event name: `s3-geojson-test`
   - Template: Choose "Amazon S3 Put" from dropdown

3. **Modify the Test Event**
   Replace the default test event with:
   ```json
   {
     "Records": [
       {
         "s3": {
           "bucket": {
             "name": "wwrr-return-data"
           },
           "object": {
             "key": "wwrr-coverage/US/Output Files/Geodata/US_Returns_Store_Mapping.geojson"
           }
         }
       }
     ]
   }
   ```

4. **Run the Test**
   - Click "Test" button
   - Watch the results in real-time
   - Check logs in the "Execution results" tab

**Expected Result**: Status 200 with message "Successfully processed 1 files"

#### Option 2: S3 Event Simulation (Triggers Automatic Pipeline)

```bash
# Method A: Copy to different name (creates new map)
aws s3 cp \
    s3://wwrr-return-data/wwrr-coverage/US/Output\ Files/Geodata/US_Returns_Store_Mapping.geojson \
    s3://wwrr-return-data/wwrr-coverage/US/Output\ Files/Geodata/test_trigger.geojson

# Method B: Force overwrite with metadata (updates existing map)
aws s3 cp \
    s3://wwrr-return-data/wwrr-coverage/US/Output\ Files/Geodata/US_Returns_Store_Mapping.geojson \
    s3://wwrr-return-data/wwrr-coverage/US/Output\ Files/Geodata/US_Returns_Store_Mapping.geojson \
    --metadata-directive REPLACE \
    --metadata test=trigger

# Both methods will automatically trigger Lambda and create HTML maps
```

### Monitor Lambda Execution

```bash
# Check if HTML map was generated
aws s3 ls s3://wwrr-return-data/wwrr-coverage/US/Output\ Files/Geodata/ --recursive --human-readable

# View Lambda logs
aws logs tail /aws/lambda/geojson-map-generator --follow

# Check function status
aws lambda get-function --function-name geojson-map-generator
```

### Expected Output Files

After ECS completes and Lambda triggers:
```
s3://wwrr-return-data/wwrr-coverage/US/Output Files/Geodata/
├── US_Returns_Store_Mapping.geojson          # Created by ECS
└── US_Returns_Store_Mapping_map.html         # Created by Lambda
```

### Interactive Map Features
The generated HTML map includes:
- 🔵 **Units (demand)** layer with size-based circles
- 🔴 **Total store count** layer  
- 🟡🟣🟢 **Individual partner** layers (auto-detected)
- ✅ **Toggle controls** to show/hide layers
- 📊 **Interactive legend**
- 🗺️ **Dark theme base map**
- 💬 **Hover tooltips** with data details

---

## Summary

Your coverage analysis is now running on a robust, scalable ECS platform that:

- 🗓️ **Runs automatically** every Monday at 2 AM PST
- 🚀 **Runs on-demand** with multiple manual trigger options
- 🗺️ **Auto-generates maps** via Lambda when GeoJSON is created
- 💪 **Handles large datasets** with 4 vCPU and 16GB RAM
- ⏱️ **No timeout limits** - can run as long as needed
- 🔄 **Easy updates** - just change Python file and redeploy
- 💰 **Cost effective** - ~$3/month for weekly runs + ~$0.58 per manual run + ~$0.01 per map generation
- 📊 **Reliable output** to S3 for downstream consumption

### Quick Reference for Manual Operations:

#### ECS Manual Runs:
- **Console Method**: ECS → Clusters → coverage-analysis-cluster → Run new Task
- **CLI Method**: `./manual-run.sh` (after creating the script)
- **Monitor**: CloudWatch Logs → /ecs/coverage-analysis → Latest log stream

#### Lambda Map Generation:
- **Code Updates**: Edit → Upload → `./update-lambda.sh`
- **Manual Trigger**: `aws s3 cp existing.geojson s3://bucket/trigger.geojson`
- **Monitor**: `./manage-lambda.sh logs`

For any Python script changes, simply update the file and run `./deploy.sh`. The infrastructure handles the rest automatically!
--
-

## CloudFront HTTPS HTML Hosting

### Overview
Secure HTTPS hosting solution for serving HTML files (maps, reports, dashboards) from S3 using CloudFront CDN.

**Setup Date**: August 20, 2026  
**Architecture**: S3 → CloudFront → HTTPS URLs  
**Purpose**: Professional HTTPS access to coverage maps and HTML reports  

### 🗂️ Files Used
- **`cloudfront-s3-html-hosting.yaml`** - CloudFormation template for CloudFront distribution

### 🚀 Initial Deployment Steps (COMPLETED)

#### 1. Deploy CloudFront Distribution
```bash
aws cloudformation deploy \
    --template-file cloudfront-s3-html-hosting.yaml \
    --stack-name html-hosting \
    --capabilities CAPABILITY_IAM \
    --region us-west-2
```

#### 2. Fix S3 Bucket Policy (Access Denied Resolution)
```bash
aws s3api put-bucket-policy --bucket wwrr-return-data --policy '{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::wwrr-return-data/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::883765745396:distribution/E25O3ZE140XAA0"
        }
      }
    }
  ]
}'
```

#### 3. Get CloudFront Domain
```bash
aws cloudformation describe-stacks \
    --stack-name html-hosting \
    --query 'Stacks[0].Outputs' \
    --output table
```

### ✅ Successfully Deployed Resources

#### CloudFront Distribution Details
- **Distribution ID:** `E25O3ZE140XAA0`
- **Domain:** `https://d228vtr56drz7v.cloudfront.net`
- **Stack Name:** `html-hosting`
- **Region:** `us-west-2`
- **Status:** ✅ **Successfully Deployed and Working**

#### Configuration Features
- **Source Bucket:** `wwrr-return-data`
- **HTTPS Only:** ✅ (redirects HTTP to HTTPS)
- **Global CDN:** ✅ (US/Europe/Asia regions for cost optimization)
- **Caching Strategy:**
  - HTML files: No caching (always fresh content)
  - CSS/JS/Images: Optimized caching for performance
- **Origin Access Control:** ✅ Secure S3 access
- **Cost Optimized:** PriceClass_100 (~$1-5/month)

### 🌐 Access URLs

#### Base URL Pattern
```
https://d228vtr56drz7v.cloudfront.net/[path-to-file]
```

#### Coverage Analysis Map (Primary Use Case)
```
https://d228vtr56drz7v.cloudfront.net/wwrr-coverage/US/Output%20Files/Geodata/US_Returns_Store_Mapping_map.html
```

#### Other HTML Files Examples
```
https://d228vtr56drz7v.cloudfront.net/reports/dashboard.html
https://d228vtr56drz7v.cloudfront.net/maps/store-locations.html
https://d228vtr56drz7v.cloudfront.net/any/path/file.html
```

### 🔄 Integration with ECS Pipeline

The CloudFront HTTPS hosting integrates seamlessly with the existing ECS coverage analysis pipeline:

1. **ECS Task** → Generates coverage data → Saves to S3
2. **Lambda Function** → Detects GeoJSON → Creates HTML map → Saves to S3  
3. **CloudFront** → Serves HTML map via HTTPS → Professional access

**Complete Flow**: 
```
Scheduled/Manual ECS Run → Coverage Analysis → GeoJSON Output → Lambda Map Generation → HTML Map → HTTPS Access via CloudFront
```

### 🎯 Key Benefits

- ✅ **Professional HTTPS URLs** for sharing coverage maps
- ✅ **Global CDN Performance** - fast loading worldwide
- ✅ **Always Fresh Maps** - HTML files never cached
- ✅ **Secure Access** - HTTPS encryption for all content
- ✅ **Cost Effective** - Optimized for regional distribution
- ✅ **Easy Integration** - Works with existing S3 output structure
- ✅ **Future Ready** - Any new HTML files automatically get HTTPS access

### 📊 CloudFront Management Commands

#### Check CloudFront Status
```bash
# View distribution details
aws cloudformation describe-stacks \
    --stack-name html-hosting \
    --query 'Stacks[0].Outputs' \
    --output table

# Check distribution directly
aws cloudfront get-distribution --id E25O3ZE140XAA0
```

#### Update CloudFront (If Needed)
```bash
# Update CloudFront configuration
aws cloudformation deploy \
    --template-file cloudfront-s3-html-hosting.yaml \
    --stack-name html-hosting \
    --capabilities CAPABILITY_IAM \
    --region us-west-2
```

#### Emergency CloudFront Controls
```bash
# Disable distribution (stops serving content)
aws cloudfront update-distribution \
    --id E25O3ZE140XAA0 \
    --distribution-config "$(aws cloudfront get-distribution-config --id E25O3ZE140XAA0 --query 'DistributionConfig' | jq '.Enabled = false')"

# Enable distribution (resume serving content)
aws cloudfront update-distribution \
    --id E25O3ZE140XAA0 \
    --distribution-config "$(aws cloudfront get-distribution-config --id E25O3ZE140XAA0 --query 'DistributionConfig' | jq '.Enabled = true')"
```

### 🔧 Template Configuration

The `cloudfront-s3-html-hosting.yaml` template includes:
- **CloudFront Distribution** with Origin Access Control
- **Conditional S3 Bucket Policy** (skips if policy already exists)
- **Multiple Cache Behaviors** for different file types:
  - `*.html` → No caching (always fresh)
  - `*.css`, `*.js` → Optimized caching  
  - `*.png`, `*.jpg` → Image caching
- **HTTPS-Only Protocol** policy
- **Error Page Configuration** (404 → custom error page)
- **Cost-Optimized Price Class** (PriceClass_100)

### 💡 Usage in Coverage Analysis Workflow

#### After ECS Completes (Scheduled Monday 2 AM PST or Manual Run):
1. **Check ECS Output**: Coverage data files created in S3
2. **Verify Lambda Map**: HTML map auto-generated in S3  
3. **Access via HTTPS**: Use CloudFront URL for professional sharing
4. **Share URL**: 
   ```
   https://d228vtr56drz7v.cloudfront.net/wwrr-coverage/US/Output%20Files/Geodata/US_Returns_Store_Mapping_map.html
   ```

#### Expected Files Available via HTTPS:
- **Interactive Coverage Map**: `US_Returns_Store_Mapping_map.html`
- **Any Future HTML Reports**: Automatically accessible via HTTPS
- **Supporting Assets**: CSS, JS, images served with optimized caching

### 📝 Notes & Best Practices

- **Template Safety**: Defaults to `SkipBucketPolicy=true` to avoid conflicts
- **Propagation Time**: CloudFront changes take 2-3 minutes to propagate globally
- **Cache Invalidation**: HTML files never cached, so updates appear immediately
- **Security**: Origin Access Control ensures only CloudFront can access S3 files
- **Monitoring**: CloudFront metrics available in CloudWatch
- **Custom Domain**: Can be added later if needed (requires SSL certificate)

### 🔗 Related Components

#### CloudFormation Stacks:
- **`coverage-analysis-ecs`** - ECS infrastructure for coverage analysis
- **`lambda-map-generator-stack`** - Lambda function for HTML map generation  
- **`html-hosting`** - CloudFront distribution for HTTPS serving

#### S3 Structure Integration:
```
s3://wwrr-return-data/
└── wwrr-coverage/US/Output Files/
    ├── Geodata/
    │   ├── US_Returns_Store_Mapping.geojson    # ECS output
    │   └── US_Returns_Store_Mapping_map.html   # Lambda output → CloudFront serves
    ├── Demand Coverage US Monthly.csv000       # ECS output  
    ├── Density Coverage US Monthly.csv000      # ECS output
    └── US Returns Store Mapping.csv000         # ECS output
```

**Complete Pipeline**: ECS Analysis → Lambda Map → CloudFront HTTPS → Professional Access ✅

---

## GitHub Actions: Automated Coverage Maps Publishing

### Overview
GitHub Actions workflow that automatically downloads HTML maps from CloudFront, applies dashboard styling, and publishes them to GitHub Pages for public access.

**Repository**: `https://github.com/ablash-src/coverage-maps`  
**GitHub Pages URL**: `https://ablash-src.github.io/coverage-maps/`  
**Workflow File**: `.github/workflows/update-maps.yml`  
**Schedule**: Every Tuesday at 8 AM UTC (1 AM PST)  

### 🔄 Complete Pipeline Flow

```
ECS Analysis → S3 GeoJSON → Lambda HTML → CloudFront → GitHub Actions → GitHub Pages
   Monday         Monday       Monday        Tuesday         Public Access
   2 AM PST      2 AM PST     2 AM PST      1 AM PST         24/7
```

### 🏗️ Architecture Overview

#### **Pipeline Components**:
1. **ECS Coverage Analysis** (Monday 2 AM PST) → Generates GeoJSON data
2. **Lambda Map Generator** (Monday 2 AM PST) → Creates raw HTML map  
3. **CloudFront CDN** → Serves HTML via HTTPS with global distribution
4. **GitHub Actions** (Tuesday 1 AM PST) → Downloads, styles, and publishes
5. **GitHub Pages** → Public hosting at `ablash-src.github.io/coverage-maps`

#### **Key Benefits**:
- ✅ **Professional Public URL** - Easy sharing with stakeholders
- ✅ **Automated Weekly Updates** - No manual intervention needed  
- ✅ **Dashboard Styling** - Corporate "Airwolf" theme wrapping
- ✅ **Free Hosting** - GitHub Pages at no cost
- ✅ **Version Control** - All updates tracked in Git history
- ✅ **Reliability** - Multiple hosting redundancy (CloudFront + GitHub)

### 📁 GitHub Repository Structure

```
coverage-maps/
├── .github/workflows/
│   └── update-maps.yml                    # GitHub Actions workflow
├── US_Returns_Store_Mapping_map.html      # Styled dashboard version  
├── us_map_raw.html                        # Raw map for full-screen
├── index.html                             # Repository landing page
├── build_dashboard.py                     # Dashboard builder script
├── README.md                              # Repository documentation
└── [Other regional maps...]               # DE, ES, FR, IT, UK maps
```

### 🤖 GitHub Actions Workflow Details

#### **Workflow Configuration** (`.github/workflows/update-maps.yml`)

**Triggers**:
- **Scheduled**: Every Tuesday at 8 AM UTC (1 AM PST) via cron
- **Manual**: Can be triggered from GitHub Actions UI using `workflow_dispatch`

**Runner Environment**: `ubuntu-latest` (GitHub-hosted)

#### **Workflow Steps Breakdown**:

##### **Step 1: Repository Checkout**
```yaml
- name: Checkout repository
  uses: actions/checkout@v4
```
- Downloads the current repository state
- Ensures workflow has access to existing files

##### **Step 2: Download Latest Map from CloudFront**
```yaml
- name: Download map HTML from CloudFront
  run: |
    curl -sL "https://d228vtr56drz7v.cloudfront.net/wwrr-coverage/US/Output%20Files/Geodata/US_Returns_Store_Mapping_map.html" -o map_content.html
```
- **Purpose**: Gets the freshest HTML map generated by Lambda
- **Source**: CloudFront CDN (always up-to-date)
- **Method**: `curl` command for reliable HTTP download
- **Output**: `map_content.html` (temporary file)

##### **Step 3: Apply Dashboard Styling**
```yaml
- name: Build dashboard page
  run: |
    python3 -c "
    # Inline Python script that:
    # 1. Reads downloaded map_content.html
    # 2. Wraps it in corporate dashboard theme
    # 3. Adds navigation and branding elements
    # 4. Saves as US_Returns_Store_Mapping_map.html
    "
```

**Dashboard Features Applied**:
- 🎨 **"Airwolf" Corporate Theme** - Professional header with animated elements
- 🏢 **Branding** - "Partnerships Analytics Team" title and subtitle
- 📊 **Interactive Badge** - "Interactive Map" indicator  
- 🖱️ **Navigation Links** - "Open Full Screen" link to raw version
- 🎯 **Responsive Design** - Works on desktop and mobile
- 🌈 **Visual Elements** - Animated header stripes and gradients

**Python Script Functions**:
1. **Read Raw Map**: Loads `map_content.html` with UTF-8 encoding
2. **Build Header**: Creates corporate-styled header with animated elements
3. **Wrap Content**: Embeds raw map content in styled container  
4. **Add Footer**: Includes branding and navigation elements
5. **Save Dashboard**: Writes final styled version to repository

##### **Step 4: Create Full-Screen Version**
```yaml
- name: Also save raw map for full screen link
  run: |
    cp map_content.html us_map_raw.html
```
- **Purpose**: Provides clean full-screen version without dashboard wrapper
- **Usage**: Accessed via "Open Full Screen" button in dashboard
- **Benefit**: Users can view map without corporate styling if needed

##### **Step 5: Cleanup Temporary Files**
```yaml
- name: Clean up temp file
  run: rm map_content.html
```
- **Purpose**: Removes temporary download file
- **Keeps Repository Clean**: Only final files committed to Git

##### **Step 6: Commit and Push Changes**
```yaml
- name: Commit and push changes
  run: |
    git config user.name "github-actions[bot]"
    git config user.email "github-actions[bot]@users.noreply.github.com"
    git add US_Returns_Store_Mapping_map.html us_map_raw.html
    git diff --cached --quiet || (git commit -m "Auto-update coverage map $(date +%Y-%m-%d)" && git push)
```

**Git Operations**:
1. **Configure Bot Identity**: Sets GitHub Actions bot as committer
2. **Stage Files**: Adds both styled and raw HTML files  
3. **Check for Changes**: `git diff --cached --quiet` prevents empty commits
4. **Conditional Commit**: Only commits if files actually changed
5. **Auto-Push**: Immediately pushes changes to repository
6. **Timestamped Messages**: Commit message includes current date

### 🎨 Dashboard Styling Details

#### **Corporate "Airwolf" Theme Components**:

##### **Header Design**:
- **Background**: Dark blue gradient (`#0a1628`) with animated stripe overlays
- **Animated Elements**: 
  - 5 animated stripes with different opacities and gradients
  - Skewed transformation (`skewX(-20deg)`) for dynamic appearance
  - Multiple blue tones (#1a5bb5, #2d7be6, #3388ff) for depth
- **Typography**: 
  - Main title: 32px, bold, white with text shadow
  - Subtitle: 14px, semi-transparent white
  - Professional font stack: 'Segoe UI', system fonts

##### **Container Styling**:
- **Layout**: Max-width 1600px, centered with responsive padding
- **Card Design**: White background, rounded corners, subtle shadows
- **Section Headers**: Clean typography with interactive badges
- **Navigation**: Professional link styling with hover effects

##### **Interactive Elements**:
- **Map Container**: Minimum 700px height for optimal viewing
- **Full Screen Button**: Styled as professional link with hover states
- **Footer**: Centered branding with consistent typography

### 🔧 Manual Workflow Management

#### **Trigger Manual Update**:

**Method 1: GitHub Actions UI** (Recommended)
1. Go to `https://github.com/ablash-src/coverage-maps`
2. Click **"Actions"** tab
3. Select **"Update Coverage Maps"** workflow
4. Click **"Run workflow"** button (right side)
5. Click **"Run workflow"** (green button)
6. Monitor progress in real-time

**Method 2: GitHub CLI** (For Developers)
```bash
# Trigger manual run via CLI
gh workflow run update-maps.yml --repo ablash-src/coverage-maps

# Check workflow status
gh run list --repo ablash-src/coverage-maps --workflow update-maps.yml
```

#### **Recent Updates**:
- **August 2026**: Updated navigation link to point to CloudFront for real-time data access
- **Files Modified**: `.github/workflows/update-maps.yml` and `build_dashboard.py`
- **Impact**: Full Screen button now provides 24-hour fresher data than GitHub-hosted version

#### **Monitor Workflow Execution**:

**GitHub Actions UI**:
- **Live Progress**: See each step executing in real-time
- **Logs**: Click on any step to see detailed output
- **Duration**: Typical run time is 30-60 seconds
- **Success Indicators**: Green checkmarks for each completed step

**Expected Log Output**:
```bash
# Step 2: Download map HTML from CloudFront
curl output: HTTP/200, file downloaded successfully

# Step 3: Build dashboard page  
Dashboard built successfully

# Step 6: Commit and push changes
[main 1a2b3c4] Auto-update coverage map 2026-08-20
 2 files changed, 1247 insertions(+), 1203 deletions(-)
```

### 🌐 Access URLs and Usage

#### **Public Access URLs**:

**Primary Dashboard** (Styled Version):
```
https://ablash-src.github.io/coverage-maps/US_Returns_Store_Mapping_map.html
```

**Full Screen Version** (Real-Time via CloudFront):
```  
https://d228vtr56drz7v.cloudfront.net/wwrr-coverage/US/Output%20Files/Geodata/US_Returns_Store_Mapping_map.html
```

**Repository Landing Page**:
```
https://ablash-src.github.io/coverage-maps/
```

#### **📝 Navigation Link Update (August 2026)**:
Updated the "Open Full Screen" button to link directly to CloudFront instead of the GitHub-hosted raw version:

**Previous Link**: `us_map_raw.html` (GitHub Pages - 24 hours delayed)  
**Updated Link**: `https://d228vtr56drz7v.cloudfront.net/...` (CloudFront CDN - Real-time)

**Benefits**:
- ✅ **24-hour fresher data** - CloudFront updates Monday 2 AM PST vs GitHub Tuesday 1 AM PST
- ✅ **Real-time access** - Direct link to source of truth  
- ✅ **Better performance** - Global CDN optimization
- ✅ **No version discrepancies** - Consistent data between dashboard and full-screen

#### **Comparison: CloudFront vs GitHub Pages**:

| Feature | CloudFront URL | GitHub Pages URL |
|---------|----------------|------------------|
| **Purpose** | AWS Internal/Development + Real-time Full Screen | Public Stakeholder Sharing |
| **Styling** | Raw map only | Corporate dashboard theme |
| **URL Format** | Complex AWS domain | Clean github.io domain |
| **Updates** | Real-time (Monday 2 AM PST) | Weekly batch (Tuesday 1 AM PST) |
| **Audience** | Technical teams + Full-screen users | Business stakeholders |
| **Access** | Direct API access + Full-screen link | Professional presentation |
| **Navigation Link** | ✅ **Full Screen target** (as of Aug 2026) | Dashboard host |

### 📊 Workflow Monitoring and Troubleshooting

#### **Success Indicators**:
- ✅ **GitHub Actions Badge**: Green status in repository  
- ✅ **Updated Files**: New commit in repository history
- ✅ **Functional Map**: Interactive map loads at GitHub Pages URL
- ✅ **Fresh Data**: Map shows latest coverage analysis results

#### **Common Issues and Solutions**:

##### **Issue 1: Download Fails from CloudFront**
**Symptoms**: `curl` command fails in Step 2
**Causes**: 
- CloudFront distribution disabled
- Lambda hasn't generated new HTML yet  
- Network connectivity issues

**Solutions**:
```bash
# Check CloudFront status
aws cloudfront get-distribution --id E25O3ZE140XAA0

# Verify HTML file exists
curl -I "https://d228vtr56drz7v.cloudfront.net/wwrr-coverage/US/Output%20Files/Geodata/US_Returns_Store_Mapping_map.html"

# Manual Lambda trigger if needed
aws lambda invoke --function-name geojson-map-generator --payload '{...}'
```

##### **Issue 2: No Changes to Commit**
**Symptoms**: Workflow completes but no new commit
**Cause**: Downloaded HTML identical to existing version
**Result**: This is normal behavior - no unnecessary commits created

##### **Issue 3: Dashboard Styling Breaks**
**Symptoms**: Map displays but styling is corrupted
**Cause**: Python inline script error in Step 3
**Solution**: Check Python syntax in workflow YAML

##### **Issue 4: GitHub Pages Not Updating**
**Symptoms**: Repository updated but GitHub Pages shows old version
**Solutions**:
- **Wait 2-3 minutes**: GitHub Pages has deployment delay
- **Check Pages Status**: Repository → Settings → Pages → Check deployment status
- **Force Refresh**: Hard refresh browser cache (Ctrl+F5)

#### **Workflow Debugging Commands**:

```bash
# View recent workflow runs
gh run list --repo ablash-src/coverage-maps --limit 5

# Get detailed logs for specific run
gh run view [RUN_ID] --repo ablash-src/coverage-maps --log

# Download workflow logs locally
gh run download [RUN_ID] --repo ablash-src/coverage-maps
```

### 🔄 Integration with Coverage Analysis Pipeline

#### **Timing Coordination**:
- **Monday 2 AM PST**: ECS runs coverage analysis → New data available
- **Monday 2 AM PST**: Lambda generates HTML map → CloudFront serves fresh content  
- **Tuesday 1 AM PST**: GitHub Actions downloads → Applies styling → Publishes
- **Tuesday 1:01 AM PST**: Updated map available on GitHub Pages

#### **Data Flow Verification**:

**After Monday ECS Run**:
1. ✅ **Check ECS Completion**: CloudWatch logs show successful run
2. ✅ **Verify Lambda Trigger**: HTML file updated in S3 
3. ✅ **Confirm CloudFront**: Raw map accessible via HTTPS URL

**After Tuesday GitHub Actions**:
1. ✅ **Check Workflow Status**: Green badge in GitHub Actions
2. ✅ **Verify Commit**: New commit with timestamp in repository  
3. ✅ **Test Public Access**: Dashboard loads at GitHub Pages URL
4. ✅ **Validate Data**: Map shows fresh coverage analysis results

#### **End-to-End Validation**:

```bash
# 1. Check latest ECS task completion
aws ecs list-tasks --cluster coverage-analysis-cluster --desired-status STOPPED | head -5

# 2. Verify CloudFront has fresh content
curl -I "https://d228vtr56drz7v.cloudfront.net/wwrr-coverage/US/Output%20Files/Geodata/US_Returns_Store_Mapping_map.html"

# 3. Check GitHub Actions workflow status  
gh run list --repo ablash-src/coverage-maps --workflow update-maps.yml --limit 1

# 4. Test public GitHub Pages access
curl -I "https://ablash-src.github.io/coverage-maps/US_Returns_Store_Mapping_map.html"
```

### 💡 Advanced Configuration Options

#### **Modify Update Schedule**:
Edit `.github/workflows/update-maps.yml`:
```yaml
on:
  schedule:
    # Current: Every Tuesday at 1 AM PST
    - cron: '0 8 * * 2'
    
    # Examples:
    # Daily at 3 AM PST: '0 11 * * *'  
    # Twice weekly (Tue/Fri): '0 8 * * 2,5'
    # Monthly (1st of month): '0 8 1 * *'
```

#### **Add Multiple Maps**:
Extend workflow to handle additional regions:
```yaml
- name: Download additional maps
  run: |
    curl -sL "https://d228vtr56drz7v.cloudfront.net/wwrr-coverage/EU/Output%20Files/Geodata/EU_Returns_Store_Mapping_map.html" -o eu_map.html
    # Apply styling...
    # Commit additional files...
```

#### **Custom Dashboard Themes**:
Modify the Python inline script in Step 3 to:
- Change color schemes
- Add custom branding elements  
- Include additional navigation
- Embed analytics tracking

### 📈 Benefits and Business Value

#### **Stakeholder Benefits**:
- 🏢 **Professional Presentation**: Corporate-styled dashboard for executive sharing
- 🌐 **Easy Access**: Clean github.io URLs that don't expose AWS infrastructure  
- 📱 **Mobile Responsive**: Dashboard works on all device types
- 🔗 **Shareable Links**: Simple URLs for email/Slack sharing
- 📊 **Always Current**: Weekly updates ensure data freshness

#### **Technical Benefits**:
- 🤖 **Full Automation**: Zero manual intervention after setup
- 💰 **Cost Effective**: GitHub Pages hosting is free
- 🛡️ **Redundant Hosting**: Multiple access methods (CloudFront + GitHub)  
- 📝 **Version Control**: All updates tracked in Git history
- 🔍 **Transparency**: Workflow logs provide full audit trail

#### **Operational Benefits**:  
- 🕐 **Predictable Schedule**: Weekly updates every Tuesday morning
- 🚨 **Failure Notifications**: GitHub Actions emails on workflow failures
- 🔧 **Easy Maintenance**: Modify workflow via GitHub UI
- 📋 **Monitoring**: Built-in GitHub Actions dashboard for status tracking

### 🎯 Summary

The GitHub Actions workflow completes the coverage analysis pipeline by:

1. **Bridging AWS and Public Web**: Downloads from CloudFront, publishes to GitHub Pages
2. **Adding Professional Polish**: Transforms raw HTML maps into branded dashboards  
3. **Enabling Easy Sharing**: Provides clean, shareable URLs for stakeholders
4. **Maintaining Automation**: Runs weekly without manual intervention
5. **Ensuring Reliability**: Multiple hosting options and automated error handling

**Complete Automated Pipeline**: 
```
ECS (Monday 2 AM) → Lambda (Monday 2 AM) → CloudFront (Always) → GitHub Actions (Tuesday 1 AM) → GitHub Pages (24/7 Public Access)
```

This creates a fully automated, end-to-end system that takes raw coverage data and delivers it as professional, publicly accessible interactive dashboards with zero manual effort required.

---

## Lambda Console Testing (Manual Triggers)

### Overview
Easy method to test Lambda function updates directly in the AWS Console without CloudShell commands.

### 🖥️ **Lambda Console Testing (Much Easier!)**

Here's how to test your updated Lambda function directly in the AWS Console:

#### **Step 1: Navigate to Lambda Console**
- Go to **AWS Console** → **Lambda** → **Functions** → **geojson-map-generator**

#### **Step 2: Create Test Event**
- Click **"Test"** button (top right)
- **Create new test event**
- **Event name**: `s3-geojson-test`
- **Template**: Select **"Amazon S3 Put"** from dropdown

#### **Step 3: Modify Test Event JSON**
Replace the default test event with this:
```json
{
  "Records": [
    {
      "s3": {
        "bucket": {
          "name": "wwrr-return-data"
        },
        "object": {
          "key": "wwrr-coverage/US/Output Files/Geodata/US_Returns_Store_Mapping.geojson"
        }
      }
    }
  ]
}
```

#### **Step 4: Save and Test**
- Click **"Save"**
- Click **"Test"** button
- **Watch the results** in real-time!

#### **Step 5: View Results**

**In the Console, you'll see:**
- ✅ **Execution result**: Success/Failure status
- 📊 **Function logs**: Real-time output showing partner detection
- ⏱️ **Duration**: How long it took to run
- 📋 **Response**: JSON response from the function

**Expected Log Output:**
```
Scanning ALL features to detect partners with coverage...
✅ AAFES: 91 locations, 91 total stores
✅ FedEx-Stores: 1695 locations, 1813 total stores
... (all 15 partners)
Final detected partner fields: ['AAFES', 'FedEx-Stores', ...]
Total partners with coverage: 15
Uploading HTML map to: s3://wwrr-return-data/...
Successfully generated map: s3://wwrr-return-data/...
```

#### **Step 6: Verify New HTML Map**
After successful test, check the updated map:
```
https://d228vtr56drz7v.cloudfront.net/wwrr-coverage/US/Output%20Files/Geodata/US_Returns_Store_Mapping_map.html
```

### 🎯 **Console vs CloudShell Testing**

**Lambda Console Advantages:**
- ✅ **Visual interface** - see results immediately  
- ✅ **Real-time logs** - watch execution as it happens  
- ✅ **No CLI syntax** - just point and click  
- ✅ **Built-in test events** - no JSON formatting needed  
- ✅ **Instant feedback** - success/error status right there  

**CloudShell Method:**
```bash
# Alternative: Trigger via S3 copy
aws s3 cp \
    "s3://wwrr-return-data/wwrr-coverage/US/Output Files/Geodata/US_Returns_Store_Mapping.geojson" \
    "s3://wwrr-return-data/wwrr-coverage/US/Output Files/Geodata/test_trigger.geojson"

# View logs
aws logs tail /aws/lambda/geojson-map-generator --follow --since 1m
```

### 📋 **Lambda Function Updates**

#### **Current Lambda Configuration**
- **Function Name**: `geojson-map-generator`
- **Runtime**: Python 3.11
- **Trigger**: S3 ObjectCreated events on `.geojson` files in `wwrr-coverage/` prefix
- **Output**: Creates `_map.html` files with interactive coverage maps

#### **Recent Updates**
- **Enhanced Partner Detection**: Scans ALL features instead of just first feature
- **Improved Default View**: Only "Total Store Count" visible by default
- **Future-Proof**: Automatically handles any number of new partners
- **User Control**: All partners available as unchecked options for selective viewing

#### **Map Default Behavior**
- ✅ **Units (demand)**: Visible by default
- ✅ **Total Store Count**: Visible by default
- ❌ **All Partners**: Hidden by default (user must enable individually)

This provides a clean initial view showing overall demand and store coverage, while keeping all individual partner data available for selective detailed analysis.
-
--

## Multi-Analysis ECS Deployment (NEW)

### Overview
Extension of the ECS infrastructure to support three additional analysis scripts running on separate schedules.

**Deployment Date**: Current  
**Architecture**: EventBridge → ECS Fargate (Multiple Tasks) → S3  
**Scripts**: Density, Map, and Travel Percentile Analysis  
**Schedules**: Staggered weekly runs (Tue/Wed/Thu 2 AM PST)

### 📊 Analysis Schedule

| Script | Day | Time (PST) | Time (UTC) | Output File |
|--------|-----|------------|------------|-------------|
| **Coverage** | Monday | 2:00 AM | 10:00 UTC | `Demand Coverage US Monthly.csv000` |
| **Density** | Tuesday | 2:00 AM | 10:00 UTC | `Density Coverage US Monthly.csv000` |
| **Map** | Wednesday | 2:00 AM | 10:00 UTC | `US Returns Store Mapping.csv000` + `US_Returns_Store_Mapping.geojson` |
| **Travel Percentile** | Thursday | 2:00 AM | 10:00 UTC | `Travel Percentile US Monthly.csv000` |

### 🚀 Initial Deployment (One-Time Setup)

#### Files Required
```
multi-analysis/
├── us_live_density_analysis.py     # Density analysis script
├── us_live_map_analysis.py         # Map analysis script  
├── us_live_tp_analysis.py          # Travel percentile analysis script
├── ecs-multi-analysis-stack.yaml   # ECS infrastructure
├── deploy-multi-analysis.sh        # Deployment script
├── requirements.txt                # Python dependencies
└── ECS-Coverage-Analysis-Guide.md  # This documentation (updated)
```

#### Deployment Steps
```bash
# 1. Upload all files to AWS CloudShell
# 2. Make deployment script executable
chmod +x deploy-multi-analysis.sh

# 3. Run deployment
./deploy-multi-analysis.sh
```

#### What Gets Created
- **ECS Cluster**: `us-live-analysis-cluster`
- **Task Definitions**: 
  - `us-live-density-analysis` (4 vCPU, 16GB RAM)
  - `us-live-map-analysis` (2 vCPU, 8GB RAM)
  - `us-live-tp-analysis` (4 vCPU, 16GB RAM)
- **ECR Repositories**: One per analysis type
- **EventBridge Rules**: Separate schedules (Tue/Wed/Thu 2 AM PST)
- **IAM Roles**: Shared execution and task roles
- **CloudWatch Logs**: Separate log groups per analysis

### 📈 Resource Allocation

| Analysis | CPU | Memory | Complexity | Runtime |
|----------|-----|---------|------------|---------|
| **Density** | 4 vCPU | 16GB | High (complex distance calculations) | ~2-3 hours |
| **Map** | 2 vCPU | 8GB | Medium (aggregation + GeoJSON) | ~1-2 hours |
| **Travel Percentile** | 4 vCPU | 16GB | High (distance + percentile calculations) | ~2-3 hours |

### 🔧 Management Commands

#### Manual Runs (Individual)
```bash
# Density analysis
./run-density-manual.sh

# Map analysis  
./run-map-manual.sh

# Travel percentile analysis
./run-tp-manual.sh
```

#### View Logs (Individual)
```bash
# Density analysis logs
aws logs tail /ecs/us-live-density-analysis --follow

# Map analysis logs
aws logs tail /ecs/us-live-map-analysis --follow

# Travel percentile analysis logs
aws logs tail /ecs/us-live-tp-analysis --follow
```

#### Check Running Tasks
```bash
# List all tasks in analysis cluster
aws ecs list-tasks --cluster us-live-analysis-cluster

# Get task details
aws ecs describe-tasks --cluster us-live-analysis-cluster --tasks TASK_ARN
```

### 🔄 Script Updates

#### Update Individual Scripts
```bash
# 1. Edit the specific Python file (e.g., us_live_density_analysis.py)
# 2. Upload updated file to CloudShell
# 3. Re-run deployment (rebuilds only changed images)
./deploy-multi-analysis.sh
```

#### Update All Scripts
```bash
# 1. Edit multiple Python files
# 2. Upload all changed files to CloudShell  
# 3. Re-run deployment
./deploy-multi-analysis.sh
```

**Note**: The deployment script is smart enough to:
- ✅ Skip infrastructure deployment if no changes
- ✅ Rebuild only Docker images for changed scripts
- ✅ Update ECR repositories with new images
- ✅ ECS uses new images on next scheduled run

### 📊 Output Files Integration

All analysis scripts output to the same S3 structure for consistent access:

```
s3://wwrr-return-data/wwrr-coverage/US/Output Files/
├── Demand Coverage US Monthly.csv000              # Monday (Coverage)
├── Density Coverage US Monthly.csv000             # Tuesday (Density) 
├── US Returns Store Mapping.csv000                # Wednesday (Map)
├── Travel Percentile US Monthly.csv000            # Thursday (Travel Percentile)
└── Geodata/
    ├── US_Returns_Store_Mapping.geojson           # Wednesday (Map)
    └── US_Returns_Store_Mapping_map.html          # Auto-generated by Lambda
```

### 💰 Cost Analysis

#### Weekly Runs (4 analyses per week)
```
Coverage Analysis (Mon):     4 vCPU × 2h × $0.20 + 16GB × 2h × $0.09 = $1.98
Density Analysis (Tue):      4 vCPU × 2.5h × $0.20 + 16GB × 2.5h × $0.09 = $2.23  
Map Analysis (Wed):          2 vCPU × 1.5h × $0.20 + 8GB × 1.5h × $0.045 = $0.68
Travel Percentile (Thu):     4 vCPU × 2.5h × $0.20 + 16GB × 2.5h × $0.09 = $2.23
```

**Total per week**: ~$7.12  
**Total per month**: ~$28.48  
**Plus CloudWatch/ECR**: ~$2.00/month  
**Monthly total**: ~$30.48

### 🔍 Monitoring & Troubleshooting

#### Check Analysis Status
```bash
# View all EventBridge rules
aws events list-rules --name-prefix "us-live-" --region us-west-2

# Check specific rule status
aws events describe-rule --name "us-live-density-analysis-schedule" --region us-west-2
```

#### Manual Task Execution (Console Method)
1. **ECS Console**: Navigate to `us-live-analysis-cluster`
2. **Select Task Definition**: Choose appropriate analysis
3. **Run Task**: Use Fargate with public IP enabled
4. **Monitor**: Check Tasks tab for status

#### Common Issues

**Script Fails to Start**:
- Check CloudWatch logs for Python errors
- Verify S3 input data exists
- Confirm IAM roles have correct permissions

**Task Resource Limits**:
- Monitor CPU/memory usage in ECS console
- Adjust task definition resources if needed
- Consider optimizing Python scripts for efficiency

**Schedule Issues**:
- Verify EventBridge rule is enabled
- Check rule target configuration
- Test manual runs first to isolate issues

### 📅 Operational Benefits

#### Staggered Schedule Advantages
- **Resource Distribution**: Spreads compute load across week
- **Data Dependencies**: Each analysis uses most recent data
- **Troubleshooting**: Isolated failure points per analysis
- **Cost Management**: Predictable resource usage patterns

#### Individual Analysis Benefits
- **Focused Logging**: Separate log groups for easier debugging
- **Independent Scaling**: Right-size resources per workload
- **Flexible Updates**: Update one analysis without affecting others
- **Parallel Development**: Different teams can work on different analyses

### 🎯 Integration with Existing Infrastructure

The multi-analysis deployment integrates seamlessly with existing components:

- **S3 Bucket**: Same `wwrr-return-data` bucket structure
- **Lambda Map Generator**: Auto-triggers on GeoJSON from Map Analysis
- **CloudFront HTTPS**: Serves generated HTML maps
- **Existing Coverage Analysis**: Continues on Monday schedule unchanged

**Complete Weekly Flow**:
```
Mon: Coverage Analysis → Demand coverage data
Tue: Density Analysis → Partner density metrics  
Wed: Map Analysis → GeoJSON + CSV → Lambda → HTML map → HTTPS via CloudFront
Thu: Travel Percentile Analysis → Customer accessibility metrics
```

### 🚨 Emergency Operations

#### Disable All Analysis Schedules
```bash
# Disable all analysis rules
for rule in $(aws events list-rules --name-prefix "us-live-" --query 'Rules[].Name' --output text); do
    aws events disable-rule --name "$rule" --region us-west-2
done
```

#### Enable All Analysis Schedules  
```bash
# Enable all analysis rules
for rule in $(aws events list-rules --name-prefix "us-live-" --query 'Rules[].Name' --output text); do
    aws events enable-rule --name "$rule" --region us-west-2
done
```

#### Nuclear Option (Delete Everything)
```bash
# Delete the multi-analysis stack (keeps existing coverage-analysis-ecs intact)
aws cloudformation delete-stack --stack-name us-live-analysis-ecs --region us-west-2
```

### 🔗 Quick Reference

#### Stack Information
- **Stack Name**: `us-live-analysis-ecs`
- **Cluster**: `us-live-analysis-cluster`  
- **Region**: `us-west-2`
- **Account**: `883765745396`

#### Key ARNs
- **Cluster**: `arn:aws:ecs:us-west-2:883765745396:cluster/us-live-analysis-cluster`
- **Task Definitions**: 
  - `arn:aws:ecs:us-west-2:883765745396:task-definition/us-live-density-analysis`
  - `arn:aws:ecs:us-west-2:883765745396:task-definition/us-live-map-analysis`
  - `arn:aws:ecs:us-west-2:883765745396:task-definition/us-live-tp-analysis`

#### Update Workflow Summary
1. **Edit Python scripts** locally
2. **Upload to CloudShell**
3. **Run deployment**: `./deploy-multi-analysis.sh`
4. **Verify**: Check next scheduled run or manual test
5. **Monitor**: Use CloudWatch logs for troubleshooting

---

## Complete Analysis Pipeline Summary

With the multi-analysis deployment, you now have a comprehensive, automated analysis pipeline:

### 📊 **4-Day Weekly Analysis Cycle**
- **Monday**: Coverage Analysis (partner coverage metrics)
- **Tuesday**: Density Analysis (partner density/competition metrics)  
- **Wednesday**: Map Analysis (geographic visualization + CSV mapping)
- **Thursday**: Travel Percentile Analysis (customer accessibility metrics)

### 🏗️ **Infrastructure Stack**
- **Coverage Analysis**: `coverage-analysis-ecs` (existing, unchanged)
- **Multi-Analysis**: `us-live-analysis-ecs` (new deployment)
- **Map Generation**: `lambda-map-generator-stack` (existing Lambda)
- **HTTPS Hosting**: `html-hosting` (existing CloudFront)

### 📁 **Complete Output Structure**
```
s3://wwrr-return-data/wwrr-coverage/US/Output Files/
├── Demand Coverage US Monthly.csv000              # Mon: Coverage metrics
├── Density Coverage US Monthly.csv000             # Tue: Density metrics
├── US Returns Store Mapping.csv000                # Wed: Mapping data
├── Travel Percentile US Monthly.csv000            # Thu: Accessibility metrics
└── Geodata/
    ├── US_Returns_Store_Mapping.geojson           # Wed: Geographic data  
    └── US_Returns_Store_Mapping_map.html          # Wed: Interactive map
```

### 🌐 **Public Access**
Interactive map available via HTTPS:
```
https://d228vtr56drz7v.cloudfront.net/wwrr-coverage/US/Output%20Files/Geodata/US_Returns_Store_Mapping_map.html
```

### 💡 **Management Simplicity**
- **Single Update Command**: `./deploy-multi-analysis.sh` updates all analyses
- **Individual Manual Runs**: Separate scripts for each analysis
- **Centralized Monitoring**: All logs in CloudWatch, organized by analysis type
- **Cost Predictable**: ~$30/month for full 4-analysis pipeline
- **Highly Reliable**: ECS Fargate with auto-retry and comprehensive logging

**Your coverage analysis pipeline is now enterprise-ready with comprehensive geographic, density, and accessibility insights delivered weekly! 🚀**

## Map Analysis Deployment Steps

### Step 1: Deploy Map Stack (if not done yet)
```bash
chmod +x deploy-map-analysis.sh
./deploy-map-analysis.sh
```

### Step 2: Build and Push Map Analysis Docker Image

Once the stack is deployed and you get the repository URI, run:

```bash
# Create Dockerfile for map analysis
cat > Dockerfile << EOF
FROM python:3.9-slim

WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy analysis script
COPY us_live_map_analysis.py .

# Run the analysis
CMD ["python", "us_live_map_analysis.py"]
EOF

# Create requirements.txt (same as before)
cat > requirements.txt << EOF
pandas>=2.0.0
numpy>=1.24.0
boto3>=1.26.0
pytz>=2023.3
EOF

# Login to ECR 
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 883765745396.dkr.ecr.us-west-2.amazonaws.com

# Build and push the image (replace with YOUR map stack repository URI)
docker build -t us-live-map-analysis .
docker tag us-live-map-analysis:latest <YOUR_MAP_REPOSITORY_URI>:latest
docker push <YOUR_MAP_REPOSITORY_URI>:latest
```

### Step 3: Test Manual Run
```bash
# Run the ECS task manually (replace with YOUR map cluster name)
aws ecs run-task \
    --cluster <YOUR_MAP_CLUSTER_NAME> \
    --task-definition us-live-map-stack-task \
    --launch-type FARGATE \
    --network-configuration "awsvpcConfiguration={subnets=[subnet-05fdcfc59fdff54a1,subnet-043d267228cb7fec2,subnet-05073bff112cfd5ad,subnet-019b6366d96457be7],assignPublicIp=ENABLED}" \
    --region us-west-2
```

## Travel Percentile Analysis Deployment Steps

### Step 1: Deploy TP Stack
```bash
chmod +x deploy-tp-analysis.sh
./deploy-tp-analysis.sh
```

### Step 2: Build and Push Travel Percentile Docker Image
```bash
# Create Dockerfile for travel percentile analysis
cat > Dockerfile << EOF
FROM python:3.9-slim

WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy analysis script
COPY us_live_tp_analysis.py .

# Run the analysis
CMD ["python", "us_live_tp_analysis.py"]
EOF

# Create requirements.txt (same as before)
cat > requirements.txt << EOF
pandas>=2.0.0
numpy>=1.24.0
boto3>=1.26.0
pytz>=2023.3
EOF

# Login to ECR 
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 883765745396.dkr.ecr.us-west-2.amazonaws.com

# Build and push the image (replace with YOUR TP stack repository URI)
docker build -t us-live-tp-analysis .
docker tag us-live-tp-analysis:latest <YOUR_TP_REPOSITORY_URI>:latest
docker push <YOUR_TP_REPOSITORY_URI>:latest
```

### Step 3: Test Manual Run
```bash
# Run the ECS task manually (replace with YOUR TP cluster name)
aws ecs run-task \
    --cluster <YOUR_TP_CLUSTER_NAME> \
    --task-definition us-live-tp-stack-task \
    --launch-type FARGATE \
    --network-configuration "awsvpcConfiguration={subnets=[subnet-05fdcfc59fdff54a1,subnet-043d267228cb7fec2,subnet-05073bff112cfd5ad,subnet-019b6366d96457be7],assignPublicIp=ENABLED}" \
    --region us-west-2
```

## Summary

**Three Independent Stacks Created:**
1. **`us-live-density-stack`** → 4 vCPU, 16GB (intensive processing)
2. **`us-live-map-stack`** → 2 vCPU, 8GB (lighter workload)  
3. **`us-live-tp-stack`** → 4 vCPU, 16GB (intensive calculation)

**Expected Output Files:**
- `s3://wwrr-return-data/wwrr-coverage/US/Output Files/Density Coverage US Monthly.csv000`
- `s3://wwrr-return-data/wwrr-coverage/US/Output Files/US Returns Store Mapping.csv000`
- `s3://wwrr-return-data/wwrr-coverage/US/Output Files/Geodata/US_Returns_Store_Mapping.geojson`
- `s3://wwrr-return-data/wwrr-coverage/US/Output Files/Travel Percentile US Monthly.csv000`

**Each analysis runs independently with Exit code: 0 indicating success! 🎉**## Lambda F
unction Update Instructions

### Updating Lambda Function Code Outside CloudShell

When you need to update the `lambda_map_generator.py` function with changes (like API keys), follow these steps:

#### Method 1: Create ZIP File Locally

**Mac/Linux Terminal:**
```bash
# Navigate to folder containing the file
cd /path/to/your/file

# Create zip file
zip lambda-function.zip lambda_map_generator.py
```

**Windows (File Explorer):**
1. Right-click on `lambda_map_generator.py` file
2. Select "Send to" → "Compressed (zipped) folder"  
3. Rename the zip file to `lambda-function.zip`

**Windows (PowerShell):**
```powershell
# Navigate to folder containing lambda_map_generator.py
cd "C:\path\to\your\file"

# Create zip file
Compress-Archive -Path lambda_map_generator.py -DestinationPath lambda-function.zip
```

#### Method 2: Upload via AWS Lambda Console

1. **Access Lambda Console**
   - Go to AWS Console → Lambda service
   - Region: us-west-2
   - Find function: `geojson-map-generator`

2. **Upload Process**
   - Scroll to "Code source" section
   - Click "Upload from" dropdown → ".zip file"
   - Click "Upload" → Browse to `lambda-function.zip`
   - Click "Save" → Click orange "Deploy" button

3. **Verify Update**
   - Check "Last modified" timestamp updates
   - Function should show your code changes

#### Method 3: Direct Code Paste (Alternative)
1. Copy the Python code from `lambda_map_generator.py`
2. In Lambda console, scroll to code editor
3. Paste the updated code directly in browser
4. Click "Deploy"

#### Method 4: Command Line Update (CloudShell)
```bash
# If updating from CloudShell
zip lambda-function.zip lambda_map_generator.py

aws lambda update-function-code \
    --function-name geojson-map-generator \
    --zip-file fileb://lambda-function.zip \
    --region us-west-2
```

### ZIP File Verification
Before uploading, verify your ZIP contains:
- ✅ `lambda_map_generator.py` (your updated file)  
- ❌ No extra folders or files

### Expected Result
After update, generated maps will use the new functionality (e.g., Carto API styling) at:
```
https://d228vtr56drz7v.cloudfront.net/wwrr-coverage/US/Output Files/Geodata/US_Returns_Store_Mapping_map.html
```###
 Creating Lambda Triggers in AWS Console

#### Adding S3 Trigger to Lambda Function

1. **Access Lambda Function**
   - Go to AWS Console → Lambda → `geojson-map-generator`
   - Click on the function name

2. **Add Trigger**
   - Scroll up to "Function overview" section
   - Click "**+ Add trigger**" button (left side of the diagram)

3. **Configure S3 Trigger**
   - **Trigger configuration**: Select "S3"
   - **Bucket**: Choose `wwrr-return-data`
   - **Event type**: Select "All object create events" or "PUT"
   - **Prefix** (optional): `wwrr-coverage/US/Output Files/`
   - **Suffix** (optional): `.geojson`
   - **Enable trigger**: ✅ Checked

4. **Advanced Options (if needed)**
   - **Recursive invocation**: Leave unchecked
   - **Filter pattern**: Use if you want specific file patterns

5. **Save Configuration**
   - Click "**Add**" button
   - Trigger should appear in the function overview diagram

#### Common Trigger Types

**S3 Trigger Example:**
- **When**: New `.geojson` file uploaded to S3
- **Action**: Lambda generates HTML map
- **Use case**: Auto-generate maps when analysis completes

**EventBridge/CloudWatch Trigger:**
- **When**: Scheduled time (e.g., daily, weekly)  
- **Action**: Lambda runs on schedule
- **Use case**: Periodic map regeneration

**API Gateway Trigger:**
- **When**: HTTP request to API endpoint
- **Action**: Lambda processes request
- **Use case**: On-demand map generation via API

#### Trigger Configuration Example (S3)
```
Trigger: S3
├── Bucket: wwrr-return-data
├── Event: s3:ObjectCreated:*
├── Prefix: wwrr-coverage/US/Output Files/
├── Suffix: .geojson
└── Status: Enabled
```

#### Verify Trigger Setup
1. **Check Function Overview**: Trigger should show as connected
2. **Test Trigger**: Upload a test file to S3 path
3. **Monitor CloudWatch Logs**: Check `/aws/lambda/geojson-map-generator`
4. **Verify Output**: Check if HTML map is generated

#### Remove/Modify Trigger
1. **Click on trigger** in Function overview
2. **Select "Delete"** or "Configure"
3. **Modify settings** as needed
4. **Save changes**

### Troubleshooting Triggers
- **Permissions**: Ensure Lambda has S3 read permissions
- **Path matching**: Verify prefix/suffix patterns match your files
- **Event types**: Choose correct S3 events (PUT, POST, COPY)
- **Logs**: Check CloudWatch logs for execution details### Testi
ng Lambda Function

#### Method 1: Test via Lambda Console (Recommended)

1. **Access Lambda Function**
   - Go to AWS Console → Lambda → `geojson-map-generator`

2. **Create Test Event**
   - Click "**Test**" button (orange, top right)
   - **Event name**: `test-map-generation`
   - **Template**: Choose "hello-world" or "S3 Put"
   - **Event JSON**: Use basic test or S3 event simulation

3. **Basic Test Event JSON:**
```json
{
  "test": true,
  "message": "Manual test execution"
}
```

4. **S3 Event Simulation (More Realistic):**
```json
{
  "Records": [
    {
      "eventVersion": "2.1",
      "eventSource": "aws:s3",
      "eventName": "ObjectCreated:Put",
      "s3": {
        "bucket": {
          "name": "wwrr-return-data"
        },
        "object": {
          "key": "wwrr-coverage/US/Output Files/Geodata/US_Returns_Store_Mapping.geojson"
        }
      }
    }
  ]
}
```

5. **Run Test**
   - Click "**Test**" button
   - Check execution results, logs, and duration
   - Look for success/failure status

#### Method 2: Test via AWS CLI

```bash
# Basic test invocation
aws lambda invoke \
    --function-name geojson-map-generator \
    --region us-west-2 \
    --payload '{"test": true}' \
    response.json

# Check response
cat response.json

# Check if function succeeded
echo $?  # Should return 0 for success
```

#### Method 3: Test via S3 Upload (Real Trigger)

```bash
# Upload a test geojson file to trigger the Lambda
aws s3 cp existing-geojson-file.geojson s3://wwrr-return-data/wwrr-coverage/US/Output\ Files/Geodata/test-trigger.geojson

# Check if Lambda was triggered by monitoring logs
```

#### Monitoring Test Results

**View Execution Results:**
- **Console**: Test results appear immediately after clicking "Test"
- **CloudWatch Logs**: Go to CloudWatch → Log Groups → `/aws/lambda/geojson-map-generator`
- **Duration & Memory**: Check performance metrics

**What to Look For:**
- ✅ **Status Code 200**: Function executed successfully  
- ✅ **No Errors**: Clean execution logs
- ✅ **Expected Output**: Check if HTML file was generated
- ❌ **Timeout**: Function taking too long
- ❌ **Memory Issues**: Out of memory errors
- ❌ **Permission Errors**: S3 access issues

#### Verify Test Success

```bash
# Check if HTML map was generated/updated
aws s3 ls s3://wwrr-return-data/wwrr-coverage/US/Output\ Files/Geodata/ --recursive

# Check the generated map
# Look for: US_Returns_Store_Mapping_map.html with recent timestamp
```

#### Test Output Verification

1. **Check S3 Output**: Verify HTML file was created/updated
2. **Open Map URL**: Test the CloudFront link works
3. **Verify Carto Integration**: Check if dark theme is applied
4. **Check File Size**: Ensure map file isn't empty or corrupted

**Expected CloudFront URL:**
```
https://d228vtr56drz7v.cloudfront.net/wwrr-coverage/US/Output%20Files/Geodata/US_Returns_Store_Mapping_map.html
```

#### Troubleshooting Test Issues

**Common Issues:**
- **File not found**: Check S3 paths and permissions
- **Timeout**: Increase Lambda timeout (default 3 seconds)
- **Memory**: Increase memory allocation if processing large files  
- **API Key**: Verify Carto API key is working
- **S3 Permissions**: Ensure Lambda can read/write S3 bucket
---

##
 CloudFront Auto-Invalidation Setup

The Lambda function now automatically invalidates CloudFront cache when new maps are generated. This ensures users see updated maps immediately.

### Configure CloudFront Distribution ID

1. **Find your CloudFront Distribution ID**:
   ```bash
   aws cloudfront list-distributions --query 'DistributionList.Items[].{Id:Id,Domain:DomainName}'
   ```

2. **Set the environment variable in Lambda**:
   ```bash
   aws lambda update-function-configuration \
     --function-name lambda_map_generator \
     --environment Variables='{CLOUDFRONT_DISTRIBUTION_ID=YOUR_DISTRIBUTION_ID}'
   ```

   Replace `YOUR_DISTRIBUTION_ID` with your actual CloudFront distribution ID (e.g., `E1234567890ABC`).

3. **Verify the configuration**:
   ```bash
   aws lambda get-function-configuration \
     --function-name lambda_map_generator \
     --query 'Environment.Variables.CLOUDFRONT_DISTRIBUTION_ID'
   ```

### How It Works

- When a new map is uploaded to S3, the Lambda function automatically creates a CloudFront invalidation
- The invalidation targets the specific HTML file path (e.g., `/us-east-1/density_map.html`)
- Users will see the updated map immediately instead of waiting for cache expiration
- If the environment variable is not set, invalidation is skipped with a warning (Lambda still works)

### Troubleshooting CloudFront Invalidation

**Check Lambda logs**:
```bash
aws logs filter-log-events \
  --log-group-name /aws/lambda/lambda_map_generator \
  --filter-pattern "CloudFront"
```

**Common issues**:
- Missing `CLOUDFRONT_DISTRIBUTION_ID` environment variable
- Lambda IAM role lacks `cloudfront:CreateInvalidation` permission
- Invalid distribution ID format

**Add CloudFront permissions to Lambda role** (if needed):
```bash
# Get the Lambda role name
ROLE_NAME=$(aws lambda get-function --function-name lambda_map_generator --query 'Configuration.Role' --output text | cut -d'/' -f2)

# Attach CloudFront policy
aws iam attach-role-policy \
  --role-name $ROLE_NAME \
  --policy-arn arn:aws:iam::aws:policy/CloudFrontFullAccess
```
---


## CloudFront Auto-Invalidation Setup (Complete)

### Overview
The Lambda function `geojson-map-generator` automatically invalidates CloudFront cache after uploading new HTML maps to S3, ensuring users see fresh content immediately.

### Setup Steps Completed

1. **Environment Variable Configuration**:
   ```bash
   aws lambda update-function-configuration \
     --function-name geojson-map-generator \
     --environment Variables='{CLOUDFRONT_DISTRIBUTION_ID=E25O3ZE140XAA0}'
   ```

2. **IAM Permissions Added**:
   - Created policy: `LambdaCloudFrontInvalidation`
   - Granted `cloudfront:CreateInvalidation` permission on distribution E25O3ZE140XAA0
   - Attached to Lambda execution role

3. **Code Implementation**:
   - Added CloudFront client initialization
   - URL encoding for paths with spaces (`Output Files` → `Output%20Files`)
   - Automatic invalidation after S3 upload
   - Error handling with warnings if invalidation fails

### How It Works
1. Lambda uploads HTML to S3
2. Reads `CLOUDFRONT_DISTRIBUTION_ID` environment variable
3. Creates invalidation for specific HTML file path
4. Users see updated maps immediately (no cache delays)

### CloudFront Distribution
- **ID**: E25O3ZE140XAA0
- **Domain**: d228vtr56drz7v.cloudfront.net
- **Auto-invalidation**: ✅ Active

### Troubleshooting
- Check Lambda logs for "CloudFront invalidation created successfully"
- Verify environment variable is set
- Ensure IAM role has CloudFront permissions
- Path encoding handles spaces in S3 key names