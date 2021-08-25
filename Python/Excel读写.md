```python
#/usr/bin/python2
#coding:utf-8

import openpyxl
import sys
import json
import os
import re
from ast import literal_eval
excel_path = sys.argv[1]

'''
    执行方式 python xxx.py xxx.xls
    输出 output.txt
'''

book = openpyxl.load_workbook(excel_path,data_only=True)
sheet = book.get_sheet_by_name('Sheet1')
a = sheet['AG']
b = sheet['O']
c = sheet['AD']

for i in range(len(a)):
    if a[i].value == 2 or a[i].value == 1:
        if not b[i].value == None and not c[i].value == None:
            list_to_do = literal_eval(b[i].value.encode('utf-8').replace('], [', '\"],[\"').replace('xx,', ',').replace(', ','\",\"').replace('[[','[[\"').replace(']]','\"]]'))
            if re.match('^\$', list_to_do[-1][-2]) == None:
                if re.match('^\$', list_to_do[0][-2]) == None:
                    print('%s %s %s' % (i, list_to_do[0][-2], list_to_do[0][-4]))
                else:
                    sheet['AH%s' % str(i+1)] = c[i].value.replace('select ', 'select %s as %s, %s as %s,' % (list_to_do[0][-2], list_to_do[0][-4], list_to_do[1][-2], list_to_do[1][-4]))
            else:
                sheet['AH%s' % str(i+1)] = c[i].value.replace(' from', ', %s as %s, %s as %s from' % (list_to_do[-2][-2], list_to_do[-2][-4], list_to_do[-1][-2], list_to_do[-1][-4]))
book.save('info.xlsx')
```