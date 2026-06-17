# Load-balancer-and-auto-scaling-setup
I implemented AWS Application Load Balancers and Auto Scaling Groups to provide high availability and elasticity for cloud-native applications. Using Python (Boto3), I automated ALB, Launch Template, Auto Scaling Group, Scaling Policies, and CloudWatch Alarms


# Load Balancer and Auto Scaling Setup

## AWS ALB + Auto Scaling (Python, PySpark, Go)

### Architecture

```text
Users
   |
Route53
   |
Application Load Balancer (ALB)
   |
-------------------------
|         |             |
EC2-1    EC2-2       EC2-3
   |
Auto Scaling Group
   |
CloudWatch Metrics
```

---

# Python (Boto3)

## Create Launch Template

```python
import boto3

ec2 = boto3.client('ec2')

response = ec2.create_launch_template(
    LaunchTemplateName='web-template',
    LaunchTemplateData={
        'ImageId': 'ami-123456789',
        'InstanceType': 't3.micro',
        'SecurityGroupIds': ['sg-123456']
    }
)

print(response)
```

---

## Create Auto Scaling Group

```python
import boto3

asg = boto3.client('autoscaling')

asg.create_auto_scaling_group(
    AutoScalingGroupName='web-asg',
    LaunchTemplate={
        'LaunchTemplateName': 'web-template'
    },
    MinSize=2,
    MaxSize=10,
    DesiredCapacity=2,
    VPCZoneIdentifier='subnet-111,subnet-222'
)
```

---

## Scaling Policy

```python
asg.put_scaling_policy(
    AutoScalingGroupName='web-asg',
    PolicyName='cpu-scale-out',
    AdjustmentType='ChangeInCapacity',
    ScalingAdjustment=1
)
```

---

## CloudWatch Alarm

```python
cw = boto3.client('cloudwatch')

cw.put_metric_alarm(
    AlarmName='HighCPU',
    MetricName='CPUUtilization',
    Namespace='AWS/EC2',
    Statistic='Average',
    Period=300,
    Threshold=70,
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=2
)
```

---

# Create Application Load Balancer

```python
elbv2 = boto3.client('elbv2')

response = elbv2.create_load_balancer(
    Name='app-alb',
    Subnets=[
        'subnet-111',
        'subnet-222'
    ],
    SecurityGroups=['sg-123456'],
    Scheme='internet-facing',
    Type='application'
)

print(response)
```

---

# Create Target Group

```python
tg = elbv2.create_target_group(
    Name='web-target-group',
    Protocol='HTTP',
    Port=80,
    VpcId='vpc-123456'
)
```

---

# Create Listener

```python
elbv2.create_listener(
    LoadBalancerArn='alb-arn',
    Protocol='HTTP',
    Port=80,
    DefaultActions=[
        {
            'Type': 'forward',
            'TargetGroupArn': 'tg-arn'
        }
    ]
)
```

---

# PySpark Example

## Analyze Auto Scaling Logs

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("AutoScalingLogs") \
    .getOrCreate()

logs = spark.read.json(
    "s3://company-logs/autoscaling/"
)

logs.show()
```

---

## Average CPU Usage

```python
from pyspark.sql.functions import avg

cpu = logs.groupBy("instanceId") \
          .agg(avg("cpu").alias("avg_cpu"))

cpu.show()
```

---

## Detect Scaling Events

```python
events = logs.filter(
    logs.eventType == "ScaleOut"
)

events.show()
```

---

## Save Metrics

```python
cpu.write.mode("overwrite") \
   .csv("s3://reports/cpu-report")
```

---

# Go Example

## Create ALB

```go
package main

import (
    "fmt"
)

func main() {

    albName := "app-alb"

    fmt.Println(
        "Creating Load Balancer:",
        albName,
    )
}
```

---

## Auto Scaling Monitor

```go
package main

import (
    "fmt"
)

func main() {

    cpu := 82

    if cpu > 70 {

        fmt.Println(
            "Scale Out Triggered",
        )

    }

}
```

---

## Health Check Service

```go
package main

import (
    "fmt"
    "net/http"
)

func health(
    w http.ResponseWriter,
    r *http.Request,
) {

    fmt.Fprintf(w, "Healthy")

}

func main() {

    http.HandleFunc(
        "/health",
        health,
    )

    http.ListenAndServe(
        ":8080",
        nil,
    )

}
```

---

# Terraform Alternative

## ALB

```hcl
resource "aws_lb" "app" {

  name = "app-alb"

  internal = false

  load_balancer_type = "application"

  subnets = [
    aws_subnet.public1.id,
    aws_subnet.public2.id
  ]
}
```

---

## Auto Scaling Group

```hcl
resource "aws_autoscaling_group" "web" {

  min_size         = 2

  max_size         = 10

  desired_capacity = 2

  vpc_zone_identifier = [
    aws_subnet.public1.id,
    aws_subnet.public2.id
  ]
}
```

---

# Production Scaling Strategy

```text
CPU < 30%
   |
Scale In

CPU > 70%
   |
Scale Out

Min Instances = 2

Max Instances = 10
```

---

# Cost Optimization (Hinglish)

### Scale-Out

```text
Traffic Badhta Hai
↓
CPU > 70%
↓
New EC2 Launch
↓
User Experience Better
```

### Scale-In

```text
Traffic Kam Hai
↓
CPU < 30%
↓
Extra EC2 Remove
↓
AWS Cost Bachti Hai
```

### Example Savings

Without Auto Scaling:

```text
10 EC2 Always Running

Cost = ₹80,000/month
```

With Auto Scaling:

```text
2-10 EC2 Dynamic

Cost = ₹35,000–50,000/month
```

Approximate Savings:

```text
30%–60% Infrastructure Cost Reduction
```

### Interview Answer

> I implemented AWS Application Load Balancers and Auto Scaling Groups to provide high availability and elasticity for cloud-native applications. Using Python (Boto3), I automated ALB, Launch Template, Auto Scaling Group, Scaling Policies, and CloudWatch Alarms. PySpark was used to analyze scaling logs and infrastructure metrics from S3. Go services exposed health-check endpoints and scaling monitors. The solution improved application availability, enabled dynamic scaling based on CPU utilization, and reduced infrastructure costs through automated scale-in and scale-out policies.
