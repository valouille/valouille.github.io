---
title: Mettre en place du filtrage IP sur un bucket S3
author: VaLouille
type: post
date: 2018-02-28T15:20:33+00:00
url: /2018/02/mettre-en-place-du-filtrage-ip-sur-un-bucket-s3/
categories:
  - AWS

---
Pour autoriser l&rsquo;accès à un bucket S3 uniquement à partir de certaines IPs, il faut associer une bucket policy comme celle ci-dessous :

```
{
"Version": "2012-10-17",
"Id": "S3PolicyMyIpFilter",
"Statement": [
    {
        "Sid": "IPAllow",
        "Effect": "Allow",
        "Principal": "*",
        "Action": "s3:*",
        "Resource": "arn:aws:s3:::bucket-name/*",
        "Condition": {
            "IpAddress": {
                "aws:SourceIp": [
                    "123.56.78.9/32",
                    "98.76.54.32/32"
                ]
            }
        }
    }
]
}
```

Il faut adapter le nom du bucket S3, et paramétrer les IPs ou les ranges que l&rsquo;on souhaite autoriser.
