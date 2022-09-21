---
title: "How to use Terraform to deploy an Application Load-Balancer (ALB) with multiple SSL certificates"
date: 2018-03-22T12:12:08+01:00
draft: false
author: VaLouille
type: post
categories:
  - AWS
  - Terraform
tags:
  - aws
  - alb
  - ssl
  - Terraform
  - HTTPS
  - ALB
---

The *old* Elastic Load Balancer (ELB) now known as *Classic Load-Balancer* currently only supports one SSL certificate. Since October 2017, it's possible to use [up to 25 SSL certificates][1] on a single Application Load-Balancer (ALB). The ALB will then use [SNI][2] to provide the right SSL certificate depending on the URL accessed.

> AWS Provider 1.10.0 is required, included from Terraform 0.11.4, but it's possible to use it with an [older Terraform version][4]

# Deployment using Terraform

## Importing SSL certificates into IAM

### Certificate and key files

> AWS re-processes the uploaded data. Therefore, it's crucial to use the same syntax and indentation as AWS, otherwise, at each run, Terraform will query AWS to get the current certificate and think it's different, recreating it at each run. The correct syntax is explained below.

We create a `certificates` folder and place files in this directory. The syntax of the files should be like this :

* The key file :

```
-----BEGIN PRIVATE KEY-----
[...]
-----END PRIVATE KEY-----
```

* The certificate file :

```
-----BEGIN CERTIFICATE-----
[...]
-----END CERTIFICATE-----
```


* The certificate chain (if needed):

```
-----BEGIN CERTIFICATE-----
[...]
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
[...]
-----END CERTIFICATE-----
```

Do not add any new line between the certificates, or at the end of the line. If Terraform wants to recreate the certificate at each run, check the syntax of the certificate parts with the following command (`awscli` required) and compare it to your files : `aws iam get-server-certificate --server-certificate-name <xxx>`

### Upload to AWS

For every certificate we want to use, we use the following syntax :
```
resource "aws_iam_server_certificate" "url1_valouille_fr" {
  name      = "url1_valouille_fr"
  certificate_body = "${file("certificates/url1_valouille_fr.crt")}"
  private_key      = "${file("certificates/url1_valouille_fr.key")}"

}
```

If needed, we also have to add the full certificate chain of trust :
```
resource "aws_iam_server_certificate" "url2_valouille_fr" {
  name      = "url2_valouille_fr"
  certificate_body = "${file("certificates/url2_valouille_fr.crt")}"
  private_key      = "${file("certificates/url2_valouille_fr.key")}"
  certificate_chain = "${file("certificates/url2_valouille_fr.pem")}"
}

resource "aws_iam_server_certificate" "url3_valouille_fr" {
  name      = "url3_valouille_fr"
  certificate_body = "${file("certificates/url3_valouille_fr.crt")}"
  private_key      = "${file("certificates/url3_valouille_fr.key")}"
  certificate_chain = "${file("certificates/url3_valouille_fr.pem")}"
}
```

## ALB creation

### Load-balancer and targets setup

In this example, we only create one ALB and one target group, exposed over HTTPS and with two HTTP backends. For better security, it's possible to also use the SSL certificates on the backends to do end-to-end encryption (from the ALB to the EC2 instances). We then create a rule to associate the target group to the ALB, and we attach the EC2 backend instances to the target group. For more information regarding the ALB concepts, you can refer to [the official AWS page][3]

#### Creation of the ALB itself

```
resource "aws_alb" "alb_front" {
	name		=	"front-alb"
	internal	=	false
	security_groups	=	["${aws_security_group.traffic-in.id}"]
	subnets		=	["${aws_subnet.public-1a.id}", "${aws_subnet.public-1b.id}"]
	enable_deletion_protection	=	true
}
```

#### Creation of the target group

```
resource "aws_alb_target_group" "alb_front_https" {
	name	= "alb-front-https"
	vpc_id	= "${var.vpc_id}"
	port	= "443"
	protocol	= "HTTPS"
	health_check {
                path = "/healthcheck"
                port = "80"
                protocol = "HTTP"
                healthy_threshold = 2
                unhealthy_threshold = 2
                interval = 5
                timeout = 4
                matcher = "200-308"
        }
}
```
#### Assignment of the EC2 instances to the target group

```
resource "aws_alb_target_group_attachment" "alb_backend-01_http" {
  target_group_arn = "${aws_alb_target_group.alb_front_https.arn}"
  target_id        = "${aws_instance.backend-01.id}"
  port             = 80
}

resource "aws_alb_target_group_attachment" "alb_backend-02_http" {
  target_group_arn = "${aws_alb_target_group.alb_front_https.arn}"
  target_id        = "${aws_instance.backend-01.id}"
  port             = 80
}
```

#### Expose the ALB with a default certificate

It's possible to use up to 50 rules at the time of redaction. Here, we only have a default action. We also choose the default SSL certificate to use.

```
resource "aws_alb_listener" "alb_front_https" {
	load_balancer_arn	=	"${aws_alb.alb_front.arn}"
	port			=	"443"
	protocol		=	"HTTPS"
	ssl_policy		=	"ELBSecurityPolicy-2016-08"
	certificate_arn		=	"${aws_iam_server_certificate.url1_valouille_fr.arn}"

	default_action {
		target_group_arn	=	"${aws_alb_target_group.alb_front_https.arn}"
		type			=	"forward"
	}
}
```

#### Assigning the additional certificates to the ALB

For each certificate (up to 25), it's necessary to explicitly enable it on the ALB :

```
resource "aws_lb_listener_certificate" "url2_valouille_fr" {
  listener_arn    = "${aws_alb_listener.alb_front_https.arn}"
  certificate_arn = "${aws_iam_server_certificate.url2_valouille_fr.arn}"
}

resource "aws_lb_listener_certificate" "url3_valouille_fr" {
  listener_arn    = "${aws_alb_listener.alb_front_https.arn}"
  certificate_arn = "${aws_iam_server_certificate.url3_valouille_fr.arn}"
}

[...]
```

And from now, every HTTPS request made to the ALB should be served with the right as long as the corresponding certificate have been uploaded and enabled !

[1]: https://aws.amazon.com/blogs/aws/new-application-load-balancer-sni/
[2]: https://en.wikipedia.org/wiki/Server_Name_Indication
[3]: https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html
[4]: https://www.Terraform.io/docs/configuration/providers.html#provider-versions
