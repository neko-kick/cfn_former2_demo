# cfn_former2_demo
```yaml
AWSTemplateFormatVersion: "2010-09-09"
Metadata:
    Generator: "former2"
Description: ""
Resources:
    EC2Instance:
        Type: "AWS::EC2::Instance"
        Properties:
            ImageId: "ami-0404778e217f54308"
            InstanceType: "t2.micro"
            KeyName: "atsuw0-2020"
            AvailabilityZone: !Sub "${AWS::Region}a"
            Tenancy: "default"
            SubnetId: "[subneta-id]"
            EbsOptimized: false
            SecurityGroupIds: 
              - !Ref EC2SecurityGroup
            SourceDestCheck: true
            Tags: 
              - 
                Key: "Name"
                Value: "test2"
            HibernationOptions: 
                Configured: false
            EnclaveOptions: 
                Enabled: false

    EC2SecurityGroup:
        Type: "AWS::EC2::SecurityGroup"
        Properties:
            GroupDescription: "pj"
            GroupName: "pj-sgp-wordpress-002"
            Tags: 
              - 
                Key: "Name"
                Value: "pj-sgp-wordpress-002"
            VpcId: "[vpc_id]"
            SecurityGroupIngress: 
              - 
                SourceSecurityGroupId: !Sub "${ElasticLoadBalancingV2LoadBalancer.SecurityGroups}"
                SourceSecurityGroupOwnerId: !Ref AWS::AccountId
                FromPort: 80
                IpProtocol: "tcp"
                ToPort: 80
              - 
                CidrIp: "[MyIP]/32"
                FromPort: 22
                IpProtocol: "tcp"
                ToPort: 22
            SecurityGroupEgress: 
              - 
                CidrIp: "0.0.0.0/0"
                IpProtocol: "-1"

    EC2SecurityGroup2:
        Type: "AWS::EC2::SecurityGroup"
        Properties:
            GroupDescription: "pj-sgp-wordpress"
            GroupName: "pj-sgp-wordpress"
            VpcId: "[vpc_id]"
            SecurityGroupIngress: 
              - 
                CidrIp: "[MyIP]/32"
                FromPort: 80
                IpProtocol: "tcp"
                ToPort: 80
              - 
                CidrIp: "[MyIP]/32"
                FromPort: 22
                IpProtocol: "tcp"
                ToPort: 22
              - 
                CidrIp: "[MyIP]/32"
                FromPort: -1
                IpProtocol: "icmp"
                ToPort: -1
            SecurityGroupEgress: 
              - 
                CidrIp: "0.0.0.0/0"
                IpProtocol: "-1"

    ElasticLoadBalancingV2LoadBalancer:
        Type: "AWS::ElasticLoadBalancingV2::LoadBalancer"
        Properties:
            Name: "pj-test-alb"
            Scheme: "internet-facing"
            Type: "application"
            Subnets: 
              - "[subnet_c_id]"
              - "[subneta-id]"
            SecurityGroups: 
              - !Ref EC2SecurityGroup3
            IpAddressType: "ipv4"
            LoadBalancerAttributes: 
              - 
                Key: "access_logs.s3.enabled"
                Value: "false"
              - 
                Key: "idle_timeout.timeout_seconds"
                Value: "60"
              - 
                Key: "deletion_protection.enabled"
                Value: "false"
              - 
                Key: "routing.http2.enabled"
                Value: "true"
              - 
                Key: "routing.http.drop_invalid_header_fields.enabled"
                Value: "false"
              - 
                Key: "routing.http.xff_client_port.enabled"
                Value: "false"
              - 
                Key: "routing.http.desync_mitigation_mode"
                Value: "defensive"
              - 
                Key: "waf.fail_open.enabled"
                Value: "false"
              - 
                Key: "routing.http.x_amzn_tls_version_and_cipher_suite.enabled"
                Value: "false"

    ElasticLoadBalancingV2Listener:
        Type: "AWS::ElasticLoadBalancingV2::Listener"
        Properties:
            LoadBalancerArn: !Ref ElasticLoadBalancingV2LoadBalancer
            Port: 443
            Protocol: "HTTPS"
            SslPolicy: "ELBSecurityPolicy-2016-08"
            Certificates: 
              - 
                CertificateArn: !Ref CertificateManagerCertificate
            DefaultActions: 
              - 
                Order: 1
                TargetGroupArn: !Ref ElasticLoadBalancingV2TargetGroup
                Type: "forward"

    ElasticLoadBalancingV2TargetGroup:
        Type: "AWS::ElasticLoadBalancingV2::TargetGroup"
        Properties:
            HealthCheckIntervalSeconds: 30
            HealthCheckPath: "/"
            Port: 80
            Protocol: "HTTP"
            HealthCheckPort: "traffic-port"
            HealthCheckProtocol: "HTTP"
            HealthCheckTimeoutSeconds: 5
            UnhealthyThresholdCount: 2
            TargetType: "instance"
            Matcher: 
                HttpCode: "200"
            HealthyThresholdCount: 5
            VpcId: "[vpc_id]"
            Name: "pj-test-tg"
            HealthCheckEnabled: true
            TargetGroupAttributes: 
              - 
                Key: "stickiness.enabled"
                Value: "false"
              - 
                Key: "deregistration_delay.timeout_seconds"
                Value: "300"
              - 
                Key: "stickiness.app_cookie.cookie_name"
                Value: ""
              - 
                Key: "stickiness.type"
                Value: "lb_cookie"
              - 
                Key: "stickiness.lb_cookie.duration_seconds"
                Value: "86400"
              - 
                Key: "slow_start.duration_seconds"
                Value: "0"
              - 
                Key: "stickiness.app_cookie.duration_seconds"
                Value: "86400"
              - 
                Key: "load_balancing.algorithm.type"
                Value: "round_robin"
            Targets: 
              - 
                Id: !Ref EC2Instance
                Port: 80

    EC2SecurityGroup3:
        Type: "AWS::EC2::SecurityGroup"
        Properties:
            GroupDescription: "alb"
            GroupName: "pj-sgp-alb-002"
            Tags: 
              - 
                Key: "Name"
                Value: "pj-sgp-alb-002"
            VpcId: "[vpc_id]"
            SecurityGroupIngress: 
              - 
                CidrIp: "0.0.0.0/0"
                Description: "https ok"
                FromPort: 443
                IpProtocol: "tcp"
                ToPort: 443
            SecurityGroupEgress: 
              - 
                CidrIp: "0.0.0.0/0"
                IpProtocol: "-1"

    CertificateManagerCertificate:
        Type: "AWS::CertificateManager::Certificate"
        Properties:
            DomainName: "*.example.com"
            SubjectAlternativeNames: 
              - "*.example.com"
            DomainValidationOptions: 
              - 
                DomainName: "*.example.com"
                ValidationDomain: "*.example.com"
            CertificateTransparencyLoggingPreference: "ENABLED"

    Route53RecordSet:
        Type: "AWS::Route53::RecordSet"
        Properties:
            Name: "test.example.com."
            Type: "A"
            AliasTarget: 
                HostedZoneId: !GetAtt ElasticLoadBalancingV2LoadBalancer.CanonicalHostedZoneID
                DNSName: !Sub "dualstack.${ElasticLoadBalancingV2LoadBalancer.DNSName}."
                EvaluateTargetHealth: true

```


- RDS
```yml
AWSTemplateFormatVersion: "2010-09-09"

########################################
# Parameters
########################################
Parameters:
  PJName:
    Type: String
    MinLength: 1
    MaxLength: 10
    Default: test
  EnvName:
    Description: environment name
    Type: String
    AllowedValues:
      - demo
      - dev
      - stg
      - prd
    Default: dev
  SntRdsAId:
    Description: Snt Pub A Id
    Type: String
    Default: snt-xxx
  SntRdsCId:
    Description: Snt Pub C Id
    Type: String
    Default: snt-xxx
  SgpId:
    Description: Security Groups
    Type: String
    Default: "sgp-xxxxx"

########################################
# Resource
########################################
Resources:
  # DB Subnet Group
  #RDSDBSubnetGroup:
  #  Type: AWS::RDS::DBSubnetGroup
  #  Properties:
  #    SubnetIds:
  #      - !Ref "SntRdsAId"
  #      - !Ref "SntRdsCId"
  #    # DBSubnetGroupName: !Sub ${PJName}-${EnvName}-dbs-001
  DBParameterGroup:
    Type: "AWS::RDS::DBParameterGroup"
    Properties:
      # aws rds describe-db-engine-versions --query "DBEngineVersions[].DBParameterGroupFamily"
      Family: mariadb10.5
      Description: My DB Parameter Group
      Parameters:
        character_set_client: utf8mb4
        character_set_connection: utf8mb4
        character_set_database: utf8mb4
        character_set_filesystem: utf8mb4
        character_set_results: utf8mb4
        character_set_server: utf8mb4
        max_allowed_packet: 10485760
        max_connect_errors: 999999999
        slave_max_allowed_packet: 10485760
        time_zone: "Asia/Tokyo"

  RDSDBInstance:
    Type: "AWS::RDS::DBInstance"
    Properties:
      DBInstanceIdentifier: !Sub ${PJName}-${EnvName}-rds-mariadb-002
      AllocatedStorage: 20
      DBInstanceClass: "db.t3.micro"
      Engine: "mariadb"
      MasterUsername: "root"
      MasterUserPassword: "REPLACEME"
      DBName: "wordpress"
      PreferredBackupWindow: "19:00-19:30"
      BackupRetentionPeriod: 2
      PreferredMaintenanceWindow: "mon:20:00-mon:20:30"
      MultiAZ: true
      EngineVersion: "10.5.12"
      AutoMinorVersionUpgrade: false
      PubliclyAccessible: false
      StorageType: "gp2"
      Port: 3306
      CopyTagsToSnapshot: true
      DBSubnetGroupName: "default-vpc-xxxxx"
      VPCSecurityGroups:
        - "sg-xxxxx"
      MaxAllocatedStorage: 100
      DBParameterGroupName: "demo22-test-parametergroup"
      OptionGroupName: "default:mariadb-10-5"
      MonitoringRoleArn: !Sub "arn:aws:iam::${AWS::AccountId}:role/rds-monitoring-role"
      CACertificateIdentifier: "rds-ca-2019"
```

- Edp
```yaml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
    EC2VPCEndpoint:
        Type: "AWS::EC2::VPCEndpoint"
        Properties:
            VpcEndpointType: "Interface"
            VpcId: [VPC_ID]
            ServiceName: !Sub "com.amazonaws.${AWS::Region}.ssm"
            PolicyDocument: |
                {
                  "Statement": [
                    {
                      "Action": "*", 
                      "Effect": "Allow", 
                      "Principal": "*", 
                      "Resource": "*"
                    }
                  ]
                }
            SubnetIds: 
              - [SUBA_ID]
              - [SUBC_ID]
            PrivateDnsEnabled: true
            SecurityGroupIds: 
              - [SGP_ID]
```
