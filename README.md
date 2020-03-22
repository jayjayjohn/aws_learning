# aws_learning
### User Data (user data automatically ran as super user)
```
#!/bin/bash
yum update -y
yum install -y httpd.x86_64
systemctl start httpd.service
systemctl enable httpd.service
echo "accessing $(hostname -f)" > /var/www/html/index.html

```
