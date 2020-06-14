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
connection {
    type     = "ssh"
    user     = "ec2-user"
    private_key = file("C:/Users/mamatha/Downloads/mykey.pem")
    host     = aws_instance.myinstance.public_ip
  }

  provisioner "remote-exec" {
    inline = [
      "sudo yum install httpd php git -y",
      "sudo systemctl restart httpd",
      "sudo systemctl enable httpd",
    ]
  }

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
output "myos_ip" {
  value = aws_instance.myinstance.public_ip
}


resource "null_resource" "nulllocal2"  {

  connection {
    type     = "ssh"
    user     = "ec2-user"
    private_key = file("C:/Users/mamatha/Downloads/mykey.pem")
    host     = aws_instance.myinstance.public_ip
  }

provisioner "remote-exec" {
    inline = [
      "sudo mkfs.ext4  /dev/xvda",
      "sudo mount  /dev/xvda  /var/www/html",
      "sudo rm -rf /var/www/html/*",
      "sudo git clone https://github.com/mamatha67/-HybridCloud-H.git /var/www/html/"
    ]
  }

depends_on = [
    aws_volume_attachment.ebs_att,
  ]
}
resource "aws_s3_bucket""mammubucket123"{
bucket ="mammubucket123"
acl="public-read"
force_destroy=true
cors_rule{
allowed_headers=["*"]
allowed_methods=["PUT","POST"]
allowed_origins=["https://mammubucket123"]
expose_headers=["ETag"]
max_age_seconds=3000
}
}


