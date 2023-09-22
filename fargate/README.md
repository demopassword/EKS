## Fargate log routing

*`aws-observability-namespace`*.yaml

```yaml
kind: Namespace
apiVersion: v1
metadata:
  name: aws-observability
  labels:
    aws-observability: enabled
```
```
kubectl apply -f aws-observability-namespace.yaml
```
## to cloudwatch

aws-logging-cloudwatch-configmap.yaml

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: aws-logging
  namespace: aws-observability
data:
  flb_log_cw: "false"  # Set to true to ship Fluent Bit process logs to CloudWatch.
  filters.conf: |
    [FILTER]
        Name parser
        Match *
        Key_name log
        Parser crio
    [FILTER]
        Name kubernetes
        Match kube.*
        Merge_Log On
        Keep_Log Off
        Buffer_Size 0
        Kube_Meta_Cache_TTL 300s
  output.conf: |
    [OUTPUT]
        Name cloudwatch_logs
        Match   kube.*
        region ap-northeast-2
        log_group_name my-logs
        log_stream_prefix from-fluent-bit-
        log_retention_days 60
        auto_create_group true
  parsers.conf: |
    [PARSER]
        Name crio
        Format Regex
        Regex ^(?<time>[^ ]+) (?<stream>stdout|stderr) (?<logtag>P|F) (?<log>.*)$
        Time_Key    time
        Time_Format %Y-%m-%dT%H:%M:%S.%L%z
```
command
```
kubectl apply -f aws-logging-cloudwatch-configmap.yaml

curl -O https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/cloudwatchlogs/permissions.json

aws iam attach-role-policy \
--policy-arn arn:aws:iam::111122223333:policy/eks-fargate-logging-policy \
--role-name AmazonEKSFargatePodExecutionRole
```

## to opensearch

*`aws-logging-opensearch-configmap`*.yaml

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: aws-logging
  namespace: aws-observability
data:
  output.conf: |
    [OUTPUT]
      Name  es
      Match *
      Host  search-example-gjxdcilagiprbglqn42jsty66y.region-code.es.amazonaws.com
      Port  443
      Index example
      Type  example_type
      AWS_Auth On
      AWS_Region region-code
      tls   On
```
command
```
kubectl apply -f aws-logging-opensearch-configmap.yaml

curl -O https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/amazon-elasticsearch/permissions.json

aws iam create-policy --policy-name eks-fargate-logging-policy --policy-document file://permissions.json

aws iam attach-role-policy \
--policy-arn arn:aws:iam::111122223333:policy/eks-fargate-logging-policy \
--role-name AmazonEKSFargatePodExecutionRole
```

## to kinesis data firehose

*`aws-logging-firehose-configmap`*.yaml

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: aws-logging
  namespace: aws-observability
data:
  output.conf: |
    [OUTPUT]
     Name  kinesis_firehose
     Match *
     region region-code
     delivery_stream my-stream-firehose
```

command
```
kubectl apply -f aws-logging-firehose-configmap.yaml

curl -O https://raw.githubusercontent.com/aws-samples/amazon-eks-fluent-logging-examples/mainline/examples/fargate/kinesis-firehose/permissions.json

aws iam create-policy --policy-name eks-fargate-logging-policy --policy-document file://permissions.json

aws iam attach-role-policy \
--policy-arn arn:aws:iam::111122223333:policy/eks-fargate-logging-policy \
--role-name AmazonEKSFargatePodExecutionRole
```