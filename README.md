# Hello AWS

> AWS and awscli hack session

## Usage example

Create and import a keypair

```
ssh-keygen \
	-b 2048 \
	-t rsa \
	-f id_rsa \
	-q \
	-N ""
aws ec2 import-key-pair \
	--key-name my-key \
	--public-key-material "$(cat id_rsa.pub)"
```

Create and authorize a security group

```
aws ec2 create-security-group \
	--group-name EC2SecurityGroup \
	--description "Security Group for EC2 instances to allow port 22"
aws ec2 authorize-security-group-ingress \
	--group-name EC2SecurityGroup \
	--protocol tcp \
	--port 22 \
	--cidr 0.0.0.0/0
```

Create some instances

```
aws ec2 run-instances \
	--image-id ami-aa203eca \
	--key-name my-key \
	--security-groups EC2SecurityGroup \
	--instance-type t2.micro \
	--placement AvailabilityZone=us-west-1a \
	--block-device-mappings DeviceName=/dev/sdh,Ebs={VolumeSize=100} \
	--count 2
```

Terminate instances

```
aws ec2 describe-instances \
	| jq -r '.Reservations[0].Instances[].InstanceId' \
	| xargs aws ec2 terminate-instances --instance-ids
```

Terminate keys

```
aws ec2 describe-key-pairs \
	| jq -r '.KeyPairs[].KeyName' \
	| xargs -I {} aws ec2 delete-key-pair --key-name {}
```