# aws_learning
### User Data
```
#!/bin/bash
# sudo su
yum update -y
yum install -y httpd.x86_64
systemctl start httpd.service
systemctl enable httpd.service
ehco "accessing $(hostname -f)" > /var/www/html/index.html

```
