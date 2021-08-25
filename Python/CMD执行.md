```python
#!/usr/bin/python
#coding:utf-8

import subprocess
import os
import sys

class Installer(object):
    ''''''
    def cmd_exec(self, cmd):
        print 'Executing: %s' % cmd
        try:
            s = subprocess.Popen(cmd, stdin=subprocess.PIPE, stderr=sys.stderr, close_fds=True, stdout=sys.stdout, universal_newlines=True, shell=True, bufsize=1)
            s.communicate()
            print 'Executed: %s' % cmd
        except subprocess.CalledProcessError, err:
            print 'Execute fail: %s' % err.output

    def pre_define(self):
        ip = raw_input('请输入IP：')
        print '您输入的IP是：%s，将用于Rancher、Harbor的部署。' % ip
        self.ip = ip
```