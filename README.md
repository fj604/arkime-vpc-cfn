# moloch-vpc-cfn
CloudFormation template for Moloch deployment in a dedicated VPC
--
This CloudFormation template deplos a VPC with [Moloch](https://molo.ch) full packet capture instances load-balanced and auto-scaled across two Availability Zones (AZs), enhanced with [Suricata IDS](https://suricata-ids.org/), listening on UDP port 4789 (VXLAN) and storing packet captures in S3. 
You can peer this VPC to another one running EC2 instances and create [filters](https://docs.aws.amazon.com/vpc/latest/mirroring/traffic-mirroring-filter.html) and [mirroring sessions](https://docs.aws.amazon.com/vpc/latest/mirroring/traffic-mirroring-session.html) to monitor traffic from your EC2 instances in peered VPCs using a web interface.
