# moloch-vpc-cfn
CloudFormation template for Moloch deployment in a dedicated VPC
-
This CloudFormation template deploys a VPC with [Moloch](https://molo.ch) full packet capture instances load-balanced and auto-scaled across two Availability Zones (AZs), enhanced with [Suricata IDS](https://suricata-ids.org/), listening on UDP port 4789 (VXLAN) and storing packet captures in S3. 
You can peer this VPC to another one running EC2 instances and create [filters](https://docs.aws.amazon.com/vpc/latest/mirroring/traffic-mirroring-filter.html) and [mirroring sessions](https://docs.aws.amazon.com/vpc/latest/mirroring/traffic-mirroring-session.html) to monitor traffic from your EC2 instances in peered VPCs using a web interface.

## Architecture

The VPC has 4 subnets across 2 AZs - one public and one private subnet in each AZ. This is done to ensure availability in case of one AZ failure.

Each public subnet has an Application Load Balancer listener for the viewer (Web UI) and a NAT Gateway to allow instances to download updates for Suricata.

Each private subnet has the following components:
1. Viewer EC2 instance has Moloch installed but only molochviewer service will be running. These instances are Auto-Scaled without a scaling policy. This is done to ensure that the minimum number of viewer instances are always available. The Application Load Balancer runs health checks against these instances and directs user traffic to healthy instances. Auto Scaling will terminate instances that fail health checks and replace them with new ones. Viewer service logs are streamed to CLoudWatch logs. 
2. Capture EC2 instances have Moloch installed but only molochcapture service will be running. These instances are Auto-Scaled in a similar fashion to the Viewer instances. They also have Suricata IDS installed and integrated with Moloch. Suricata update runs every hour as a cron job. Moloch Capture service and Suricata service logs are streamed to CloudWatch Logs.
3. The Network Load Balancer receives mirrored traffic over VXLAN protocol from peered VPCs and directs it to the healthy capture instances. A VPC Traffic Mirror Target is set up for this Network Load Balancer. Users can create peering connections from their VPCs and direct Traffic Mirror sessions to this Mirror Target.
4. Amazon Elasticsearch service is scaled across both private subnets.
5. Each private subnet also has an Amazon S3 service endpoint. This is done so that packet captures are uploaded to S3 from capture instances and downloaded to viewer instances over private network links - this traffic is never exposed to the Internet.

Packet captures are stored in S3.

![Diagram](diagram.svg)
