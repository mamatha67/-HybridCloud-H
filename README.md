provider "aws"  {
   region   =   "ap-south-1"
   profile  =   "mymammu"
}
resource "aws_security_group""allow_tls"{
name="allow_tls"
description="allow ssh and httpd"
 ingress {
    description = "SSH from port"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
ingress {
    description = "HTTPD port"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
ingress {
    description = "local host"
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
egress{
from_port=0
to_port=0
protocol="-1"
cidr_blocks=["0.0.0.0/0"]
}
tags={
Name="allow_tls"
}
}
variable "enter_ur_keyname"{
type=string
default="mykey"
}
resource "aws_instance" "myinstance" {
  ami           = "ami-005956c5f0f757d37"
  instance_type = "t2.micro"
key_name=var.enter_ur_keyname
security_groups=["${aws_security_group.allow_tls.name}"]

  tags = {
    Name = "linuxworldos"
  }
}
resource "aws_ebs_volume" "esb2" {
  availability_zone = aws_instance.myinstance.availability_zone
  size              = 1

  tags = {
    Name = "myebs1"
  }
}
resource "aws_volume_attachment" "ebs_att" {
  device_name = "/dev/sdd"
  volume_id   = "${aws_ebs_volume.esb2.id}"
  instance_id = "${aws_instance.myinstance.id}"
  force_detach=true
}
resource "null_resource" "nulllocal1" {
       provisioner "local-exec"{
           command = "start chrome www.yahoo.com"
         }
}
