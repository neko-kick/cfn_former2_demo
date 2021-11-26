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
