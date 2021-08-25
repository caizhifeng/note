```python
#/usr/bin/python
#coding: utf-8
import json
import requests

class RancherSync(object):
    def __init__(self):
        self.session = requests.Session()

    def __login(self):
        # 登录
        loginurl = '%s/#/login' % self.server_name
        response = self.session.get(loginurl)
        data = {'username': self.username,
            'password': self.password,
            'role':'0'
        }
        loginpost = '%s/webserver/user/login.do' % self.server_name
        response = self.session.post(loginpost, data)

        #session.auth = (TOKEN.split(':')[0], TOKEN.split(':')[1])
        #session.headers.update({'Content-Type': 'application/json'})
        #session.headers.update({'Accept': 'application/json'})
        #url = 'https://%s:8443/v3/settings/server-url' % self.ip
        #res = session.get(url, verify=False)

    def sync(self):
        self.__login()
        self.__get_system_id()

if __name__ == '__main__':
    sync_obj = RancherSync()
    sync_obj.sync()
```