# This is my 1st Terraform Project
## Objective
Using Terraform (HCL), Launch an EC2 instance on AWS which can be accessed using SSH & HTTP, so that I can test that the EC2 launch is successfull and the HTTP Web is accessible.

## 1. Generate Key-Pair file which our EC2 instance can use and later can be used to SSH to our instance.

```bash
ssh-keygen -t rsa -b 2048 -f my-key
chmod 400 ./my-key
```
## 2. CreateFirstEC2Instance
 Then use below in the main.tf

 ```terraform
 provider "aws" {
  region = "us-east-2"
}

resource "aws_key_pair" "my_key" {
  key_name   = "my-key"
  public_key = file("${path.module}/my-key.pub") # Make sure this file exists
}

resource "aws_security_group" "allow_ssh_http" {
  name        = "allow_ssh_http"
  description = "Allow SSH and HTTP access"
  vpc_id      = data.aws_vpc.default.id

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

data "aws_vpc" "default" {
  default = true
}

resource "aws_instance" "web_server" {
  ami                    = "ami-0ca4d5db4872d0c28" # Amazon Linux 2 in us-east-2
  instance_type          = "t3.micro"
  key_name               = aws_key_pair.my_key.key_name
  vpc_security_group_ids = [aws_security_group.allow_ssh_http.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello World" > /var/www/html/index.html
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              EOF

  tags = {
    Name = "HelloWorldWebServer"
  }
}
```

## 3. Verify SSH Access

```bash
ssh -i ~/.ssh/my-key ec2-user@<_public-ip of EC2 Instance_>
```
  <img width="1489" height="373" alt="image" src="https://github.com/user-attachments/assets/3b15dacf-0040-4f73-84f4-2756b11224f2" />



## 4. Verify that Site works
http://_public-ip of EC2 Instance_

  <img width="760" height="227" alt="image" src="https://github.com/user-attachments/assets/9b19dcbf-cddd-4ecd-ac98-92d38d8b4715" />

