## Generate Key-Pair file which our EC2 instance can use and later can be used to SSH to our instance.

```bash
ssh-keygen -t rsa -b 2048 -f my-key
chmod 400 ./my-key
```
## CreateFirstEC2Instance
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

## Verify SSH Access

```bash
ssh -i ~/.ssh/my-key ec2-user@_public-ip of EC2 Instance_
```

## Verify that Site works
http://_public-ip of EC2 Instance_
