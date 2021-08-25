```python
#/usr/bin/python
#coding: utf-8

import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.utils import parseaddr, formataddr

import time as Time
import os, stat, sys
from datetime import datetime, date, time, timedelta

# 邮件配置
sender = 'test@123.com'
to_address = ['test@123.com']
username = 'test@123.com'
password = 'test'

# 发送邮件
def sendmail(img):
    msgRoot = MIMEMultipart('related')
    msgRoot['Subject'] = '监控日报'
    msgRoot['From'] = sender
    msgRoot['To'] = ",".join( to_address ) # 发给多人

    content = MIMEText('<html><head><style>#string{text-align:center;font-size:25px;}</style><div id="string">VPN监控面板<div></br></head><body><img src="cid:image1" alt="image1"></body></html>','html','utf-8')
    msgRoot.attach(content)
    # 获取图片
    fp = open(img, 'rb')
    msgImage = MIMEImage(fp.read())
    fp.close()
    msgImage.add_header('Content-ID', 'image1') # 该id和html中的img src对应
    msgRoot.attach(msgImage)
    smtp = smtplib.SMTP_SSL('smtphm.qiye.163.com:465')
    smtp.login(username, password)
    smtp.sendmail(sender, to_address, msgRoot.as_string())
    smtp.quit()

if __name__ == '__main__':
    sendmail(capture())
```