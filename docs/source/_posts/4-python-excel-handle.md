---
title: python 批量抓取 html 文件内数据并写入同名 excel 文档指定位置后输出 pdf
date: 2021-12-6
tags: 
- Python
- Backend
categories: notes
photos:
    - /blog/img/python.jpg
---

<br>
<!--more-->

> 需求是朋友工作中遇到的一些问题，他需要将公司生成的很多html文件里的东西给整理出来，再用公司准备的 excel 模板去整理，获取完数据后，再将每个 excel 文档的前两个 sheet 工作薄输出 pdf

> 由于文件在别机打不开, 在此只记录代码

# 环境

这里使用 anaconda3 的包，很方便，基本数据处理遇到的包都有
使用到有: `bs4`, `xlwings`, `json`, `math`, `time`, `re`, `os`

提供的 html 页面放于 `./html-doc` 下, html 解析后的元数据存于 `./html-parse-out`下, excel 输出至 `xlsb-doc` 下;

主要编辑代码在 `./init.py`内; 同时我朋友的文档做了限制只能在他本机打开, 所以就给他先配置 python 环境, 后边遇到问题是, 他本机因为 excel 打开会有登录工作账号(自动填充)的弹窗, 关闭也会有弹窗; 所以加入了 `./listener.py` 文件用 `pywin32` 的库来处理 windows 界面弹窗的自动关闭

# 编写

因为主要是处理 excel 的读写, pandas 的 dataFrame 感觉很不方便灵活, 于是就选了 xlwings

首先是准备工作, 将些公共常量先写好

```py
from bs4 import BeautifulSoup
from listener import MsgBoxListener
import xlwings as xw
import json
import math
import time
import re
import os

# 获取当前目录下的 html
html_path = './html-doc/'
xlsb_path = './xlsb-doc/'
output_path = './html-parse-output/'

# 收集
html_list = []
xlsb_list = []
stat_list = {}

# 收集表格数据
table_head_list = []
table_body_list = []
```

## main

入口函数 `main.py`

先核对是否文件按规格放入目标文件夹, 接着将所有 html 文档读取, 作以过滤, 将合规的 html 文档进行处理

```py
def main():
    # 判断目标文件夹是否存在
    if not os.path.isdir(xlsb_path[2:-1]):
        os.mkdir(r'' + xlsb_path[2:-1])
        print('请将代编辑的 excel 文档移入文件夹: ' + xlsb_path[2:-1])
        return
    if not os.path.isdir(html_path[2:-1]):
        os.mkdir(r'' + html_path[2:-1])
        print('请将代编辑的 html 文档移入文件夹: ' + html_path[2:-1])
        return

    # 获取 html 数据并写入目标文件
    html_list = os.listdir(html_path)
    finished_doc = 1
    for i in html_list:
        a, b = os.path.splitext(i)
        if b == '.html':
            parseHtml(html_path + i, a)
            print('[' + a + '] 文件已解析，当前已有' + str(finished_doc) + '个完成')
            finished_doc += 1

    print('处理完毕！')

if __name__ == '__main__':
    main()
```

## 解析 html

解析 html 函数 `parseHtml`
使用 beautifulSoup 来将 html 的目标 dom 节点给提取出来; 使用正则匹配目标信息;
其中 table 里的数据采用 二维数组 来存储, 方便之后添加进 excel, xlwings 可以以 数组 的形式向xlsx 文档添加行, 或列
最后查看当前处理的 html 名字是否和 excel 文档匹配的, 如是则进入到数据写入 xlsx 环节

```py
# 遍历每个 html
def parseHtml(file, filename):
    # 得到整个文档 dom
    doc = BeautifulSoup(open(file), from_encoding='utf-8', features='html.parser')

    # 得到时间
    time_dom = doc.find('h3')
    time_compile = re.compile('\d{4}-\d{2}-\d{2}')
    doctime = re.search(time_compile, str(time_dom))[0].replace('-', '/', 2)

    # 得到功率等级
    power_class_dom = doc.find('table')
    power_class_compile = re.compile(r'<td>(\d+)kVA')
    power_class = re.search(power_class_compile, str(power_class_dom))[1]

    # 得到设备编码 sn
    sn_number_dom = doc.find('h1')
    sn_number_compile = re.compile('\((.+)\)')
    sn_number = re.search(sn_number_compile, str(sn_number_dom))[1]

    # 得到对应 meatures 下的表格节点
    measures = doc.find('h3', string='Measures')
    table = measures.find_parent().find_next_sibling()

    # 获得表头字段
    table_head = table.find('thead').find_all('td')
    for td in table_head:
        if td.string != None:
            table_head_list.append(td.string)

    # 获得表身数据 并整理为一个对象
    table_body = table.find('thead').find_next_sibling().contents
    table_body_list = {}
    table_body_item_flag = 0
    for x in table_body:
        if re.search(re.compile('table'), str(x)):
            item = x.find_all('td')
            item_transform = {}
            # 当前 L1, L2, L3 的计数
            curr_L = []
            L_index = 1
            L_length = 3
            # 当前遍历 td 的长度
            item_length = len(item)
            # 当前遍历 td 内元素的计数
            el_length = item_length // L_length
            el_last_length = item_length - el_length * 2
            el_index = 1
            for v in item:
                v = v.string
                curr_L.append(v)
                # # 对 battery 特殊处理
                if table_body_item_flag == 3:
                    item_transform = curr_L
                elif (el_index == el_length and L_index != 3) or (L_index == 3 and el_index == el_last_length):
                    item_transform['L' + str(L_index)] = curr_L
                    curr_L = []
                    el_index = 0
                    L_index += 1
                el_index += 1
            # 数组拍平 -----
            # item_transform = curr_L
            # -------------
            table_body_list[table_head_list[table_body_item_flag]] = item_transform
            table_body_item_flag += 1
            if table_body_item_flag >= 4:
                break

    # 对表格进行补全
    export_excel = table_body_list.copy()
    for k, v in export_excel.items():
        if not isinstance(v, list):
            v['L1'].append('50 Hz')
            v['L2'].append('50 Hz')
    # 记录当前提取内容
    with open('result.txt', 'wb') as file:
        file.write(json.dumps(table_body_list, indent=4, ensure_ascii=False, sort_keys=True).encode())
    # 解析后的内容创建 excel 记录
    # createXlsx(export_excel, filename)
    # 若解析的文档存在与之对应的待编辑 xlsb 文件
    if filename in xlsb_list:
        editMatchedXlsb(table_body_list, filename, doctime, int(power_class), sn_number)
    else:
        print('提供的 excel 文档中不存在 [' + filename + ']')
```

## 针对提取出的数据写入目标 excel 特定位置

通过 xlwings 打开 excel , 再打开当前解析的 html 所对应名字的 xlsb 文件, 选择目标 sheet , 向里边具体位置写入数据, 可以区域性的横向或者纵向填入数组, 最后保存文件 wb, 关闭, 再退出 excel

```py
def editMatchedXlsb(data, filename, doctime, powerclass, snnumber):
    '''
    https://buildmedia.readthedocs.org/media/pdf/xlwings/stable/xlwings.pdf
    '''
    app = xw.App(visible=False,add_book=False)
    #不显示Excel消息框
    app.display_alerts = False
    #关闭屏幕更新,可加快宏的执行速度
    app.screen_updating = False

    wb = app.books.open(xlsb_path + filename + '.xlsb')
    # 也可以直接用 Book 创建，就不用定义 app，之后直接关闭 wb 即可
    # wb = xw.Book('test.xlsb')

    # 处理 [封面] 
    sheet1 = wb.sheets['封面']
    # 楼层
    sheet1.range('Q33').value = 'F1' if filename[0] == '1' else 'F2' if filename[0] == '2' else 'F3' if filename[0] == '3' else 'F4'
    # Site ID
    sheet1.range('Q35').value = '101' if filename[0] == '1' else '201' if filename[0] == '2' else '301' if filename[0] == '3' else '401'
    # 设备编码
    sheet1.range('Q39').value = filename
    # sn 码
    sheet1.range('Q40').value = snnumber
    # 时间
    sheet1.range('E41').value = doctime
    # 保留 stat 数据
    stat_item = []
    stat_item.append(filename) # 0 机器编码
    snnumber_item = []
    if filename[-1] == '1':
        snnumber_item.append(snnumber) # 条码为数组
        stat_item.append(snnumber_item) # 1 产品条码
        stat_item.append('1') # 2 本机编码
        stat_item.append('1') # 3 并机台数
        stat_list[filename] = stat_item
    elif filename[-1] == '2':
        # 结尾为 2 说明并机台数为 2，存在两个条码
        pre_key = filename[:-1] + '1'
        for k, v in stat_list.items():
            if k == pre_key:
                # 给前一个 1 添加，并更改
                stat_list[pre_key][1].append(snnumber)
                stat_list[pre_key][3] = '2'
                # 当前项
                snnumber_item.append(stat_list[pre_key][1][0])
                snnumber_item.append(snnumber)
                stat_item.append(snnumber_item)
                stat_item.append('2')
                stat_item.append('2')
                stat_list[filename] = stat_item
                break
    elif filename[-1] == '3':
        for i in range(2):
            crr_pre_key = filename[:-1] + str(i + 1)
            for k, v in stat_list.items():
                if k == crr_pre_key:
                    # 给前一个 1 添加，并更改
                    stat_list[crr_pre_key][1].append(snnumber)
                    stat_list[crr_pre_key][3] = '3'
                    # 当前项仅编辑一次
                    if i == 0:
                        snnumber_item.append(stat_list[crr_pre_key][1][0])
                        snnumber_item.append(stat_list[crr_pre_key][1][1])
                        snnumber_item.append(snnumber)
                        stat_item.append(snnumber_item)
                        stat_item.append('3')
                        stat_item.append('3')
                        stat_list[filename] = stat_item
                    break

    # 处理 [巡检报告]
    sheet2 = wb.sheets('巡检报告')
    # 功率等级
    sheet2.range('G33').value = powerclass
    # 电池型号
    sheet2.range('G40').value = '南都 6-GFM-180HR' if powerclass == 500 else '南都 6-GFM-155HR'
    # 电池组数
    sheet2.range('G41').value = 2 if powerclass == 300 else 3 if powerclass == 500 else 4
    # 系统类型
    sheet2.range('G35').value = '单机' if powerclass == 300 else '分散并机'
    # 外部维修旁路
    sheet2.range('G36').value = '有-3P'
    # 内部开关配置
    sheet2.range('R33').value = '主路输入、旁路、输出' if powerclass == 600 else '主路输入、旁路、输出、维修旁路'
    # 并机输出开关
    sheet2.range('R44').value = '无'
    # 时间
    sheet2.range('E462').value = doctime
    sheet2.range('Q462').value = doctime
    # 并机/本机编码/产品条码
    # 单独处理

    # Mains
    Mains = data['Mains']
    # 定义一个二维数组存储数据
    Mains_handle = [[0 for y in range(3)] for x in range(3)]
    Mains_index = 0
    # 对数据进行处理
    for k, l in Mains.items():
        for i, v in enumerate(l):
            if v[-2:] == 'Hz':
                Mains_handle[Mains_index][i] = v[:-3].strip()
            elif v[-1:] == 'V':
                Mains_handle[Mains_index][i] = round(float(v[:-1]) * math.sqrt(3), 2)
            else:
                Mains_handle[Mains_index][i] = v[:-1].strip()
        Mains_index += 1
    # 将数据渲染至目标位置 - 纵向
    sheet2.range('G167').options(transpose=True).value = Mains_handle[0]
    sheet2.range('K167').options(transpose=True).value = Mains_handle[1]
    sheet2.range('O167').options(transpose=True).value = Mains_handle[2]
    # sheet2.range('G167:R169').api.HorizontalAlignment = -4131

    # Reserve
    Reserve = data['Reserve']
    Reserve_handle = [[0 for y in range(3)] for x in range(3)]
    Reserve_index = 0
    for k, l in Reserve.items():
        # 清空二维数组初始化元素
        Reserve_handle[Reserve_index].clear()
        for i, v in enumerate(l):
            if v[-1] == 'V':
                Reserve_handle[Reserve_index].append(v[:-1].strip())
            elif v[-2:] == 'Hz':
                Reserve_handle[Reserve_index].append(v[:-2].strip())
        Reserve_index += 1
    sheet2.range('G178').options(transpose=True).value = Reserve_handle[0]
    sheet2.range('K178').options(transpose=True).value = Reserve_handle[1]
    sheet2.range('O178').options(transpose=True).value = Reserve_handle[2]
    # sheet2.range('G178:R179').api.HorizontalAlignment = -4131

    # output
    Output = data['Output']
    Output_handle = [[0 for y in range(9)] for x in range(3)]
    Output_index = 0
    for k, l in Output.items():
        for i, v in enumerate(l):
            # 两个行被清空了 ...
            Output_handle[Output_index][2] = ''
            Output_handle[Output_index][5] = ''
            if v[-1] == 'V':
                Output_handle[Output_index][0] = round(float(v[:-1].strip()) * math.sqrt(3), 2)
                Output_handle[Output_index][1] = v[:-1].strip()
            elif v[-2:] == 'Hz':
                Output_handle[Output_index][3] = v[:-2].strip()
            elif v[-1] == 'A':
                if v[-3:] == 'kVA':
                    Output_handle[Output_index][6] = v[:-3].strip()
                else:
                    Output_handle[Output_index][4] = v[:-1].strip()
            elif v[-2:] == 'PF':
                Output_handle[Output_index][7] = abs(float('0 PF'[:-2].strip()))
            else:
                Output_handle[Output_index][8] = float(l[3][:-3].strip()) * 3 / powerclass
        Output_index += 1
    sheet2.range('G263').options(transpose=True).value = Output_handle[0]
    sheet2.range('K263').options(transpose=True).value = Output_handle[1]
    sheet2.range('O263').options(transpose=True).value = Output_handle[2]
    # sheet2.range('G263:R269').api.HorizontalAlignment = -4131

    # 保存
    wb.save()
    wb.close()
    app.quit()
```

## 对数据联动处理

因为朋友的需求有个是, 得先读取完全部得表, 得到信息之后会再进行分类, 有时需要在当前表填入下个表, 或下下个表得数据, 所以在上一步收集数据后, 再写入一遍

```py
# 处理统计文件 statistics-doc
    print("\n")
    print('－－－－－－－－－－－－－－－－－－－－－－－－')
    print('进行数据整合，请等待...')
    handleStat()
    print('处理完毕！')
```

写入

```py
# 获取统计数据 stat
def handleStat():
    # 开始新的遍历获得 stat_list
    finished_doc = 1
    for v in xlsb_list:
        app = xw.App(visible=False,add_book=False)
        #不显示Excel消息框
        app.display_alerts = False
        #关闭屏幕更新,可加快宏的执行速度
        app.screen_updating = False

        wb = app.books.open(xlsb_path + v + '.xlsb')
        sheet = wb.sheets['巡检报告']

        # 并机数/本机编号
        sheet.range('R43').value = stat_list[v][2]
        sheet.range('G43').value = stat_list[v][3]
        # 并机条码
        snnumber_range = ['G55', 'R55', 'G56']
        for i, s in enumerate(stat_list[v][1]):
            sheet.range(snnumber_range[i]).value = s
        # 并机检测结果
        sheet.range('T379').value = '正常'
        sheet.range('T381').value = '正常'
        sheet.range('T382').value = '未测试'

        # 导出 pdf
        # wb.to_pdf(r'' + xlsb_path + v + '.pdf', ['封面', '巡检报告'])
        # 退出
        wb.save()
        wb.close()
        app.quit()
        print('[' + v[0] + '] 数据已整理，当前已有' + str(finished_doc) + '个完成')
        finished_doc += 1
```

## 加入关闭弹窗功能

主要逻辑是获取当前窗口得弹窗, 再获取 button 的值, 匹配 ok, 确认等值, 模拟点击

```py
import time
from threading import Thread, Event
import win32gui, win32con


class MsgBoxListener(Thread):

    def __init__(self, interval: int):
        Thread.__init__(self)
        self._interval = interval
        self._stop_event = Event()

    def stop(self): self._stop_event.set()

    @property
    def is_running(self): return not self._stop_event.is_set()

    def run(self):
        while self.is_running:
            try:
                time.sleep(self._interval)
                self._close_msgbox()
            except Exception as e:
                print(e, flush=True)

    def _close_msgbox(self):
        '''
        Click any button ("OK", "Yes" or "Confirm") to close message box.
        '''
        # get handles of all top windows
        h_windows = []
        win32gui.EnumWindows(lambda hWnd, param: param.append(hWnd), h_windows) 
    
        # check each window    
        for h_window in h_windows:            
            # get child button with text OK, Yes or Confirm of given window
            h_btn = win32gui.FindWindowEx(h_window, None,'Button', None)
            if not h_btn: continue
    
            # check button text
            text = win32gui.GetWindowText(h_btn)
            if not text.strip().lower() in ('确定', 'ok', 'yes', 'confirm'): continue
    
            # click button
            win32gui.PostMessage(h_btn, win32con.WM_LBUTTONDOWN, None, None)
            time.sleep(0.2)
            win32gui.PostMessage(h_btn, win32con.WM_LBUTTONUP, None, None)
            time.sleep(0.2)


if __name__ == '__main__':
    t = MsgBoxListener(2)
    t.start()
    time.sleep(10)
    t.stop()
```

# 综合

最后将 listen 的功能整合进主流程

```py
from bs4 import BeautifulSoup
from listener import MsgBoxListener
import xlwings as xw
import json
import math
import time
import re
import os

# 获取当前目录下的 html
html_path = './html-doc/'
xlsb_path = './xlsb-doc/'
output_path = './html-parse-output/'

stat_target = 'statistics-doc.xlsx'

# 收集
html_list = []
xlsb_list = []
stat_list = {}

# 收集表格数据
table_head_list = []
table_body_list = []

# 针对提取出的数据写入目标 excel 特定位置
def editMatchedXlsb(data, filename, doctime, powerclass, snnumber):
    '''
    https://buildmedia.readthedocs.org/media/pdf/xlwings/stable/xlwings.pdf
    '''
    app = xw.App(visible=False,add_book=False)
    #不显示Excel消息框
    app.display_alerts = False
    #关闭屏幕更新,可加快宏的执行速度
    app.screen_updating = False

    wb = app.books.open(xlsb_path + filename + '.xlsb')
    # 也可以直接用 Book 创建，就不用定义 app，之后直接关闭 wb 即可
    # wb = xw.Book('test.xlsb')

    # 处理 [封面] 
    sheet1 = wb.sheets['封面']
    # 楼层
    sheet1.range('Q33').value = 'F1' if filename[0] == '1' else 'F2' if filename[0] == '2' else 'F3' if filename[0] == '3' else 'F4'
    # Site ID
    sheet1.range('Q35').value = '101' if filename[0] == '1' else '201' if filename[0] == '2' else '301' if filename[0] == '3' else '401'
    # 设备编码
    sheet1.range('Q39').value = filename
    # sn 码
    sheet1.range('Q40').value = snnumber
    # 时间
    sheet1.range('E41').value = doctime
    # 保留 stat 数据
    stat_item = []
    stat_item.append(filename) # 0 机器编码
    snnumber_item = []
    if filename[-1] == '1':
        snnumber_item.append(snnumber) # 条码为数组
        stat_item.append(snnumber_item) # 1 产品条码
        stat_item.append('1') # 2 本机编码
        stat_item.append('1') # 3 并机台数
        stat_list[filename] = stat_item
    elif filename[-1] == '2':
        # 结尾为 2 说明并机台数为 2，存在两个条码
        pre_key = filename[:-1] + '1'
        for k, v in stat_list.items():
            if k == pre_key:
                # 给前一个 1 添加，并更改
                stat_list[pre_key][1].append(snnumber)
                stat_list[pre_key][3] = '2'
                # 当前项
                snnumber_item.append(stat_list[pre_key][1][0])
                snnumber_item.append(snnumber)
                stat_item.append(snnumber_item)
                stat_item.append('2')
                stat_item.append('2')
                stat_list[filename] = stat_item
                break
    elif filename[-1] == '3':
        for i in range(2):
            crr_pre_key = filename[:-1] + str(i + 1)
            for k, v in stat_list.items():
                if k == crr_pre_key:
                    # 给前一个 1 添加，并更改
                    stat_list[crr_pre_key][1].append(snnumber)
                    stat_list[crr_pre_key][3] = '3'
                    # 当前项仅编辑一次
                    if i == 0:
                        snnumber_item.append(stat_list[crr_pre_key][1][0])
                        snnumber_item.append(stat_list[crr_pre_key][1][1])
                        snnumber_item.append(snnumber)
                        stat_item.append(snnumber_item)
                        stat_item.append('3')
                        stat_item.append('3')
                        stat_list[filename] = stat_item
                    break

    # 处理 [巡检报告]
    sheet2 = wb.sheets('巡检报告')
    # 功率等级
    sheet2.range('G33').value = powerclass
    # 电池型号
    sheet2.range('G40').value = '南都 6-GFM-180HR' if powerclass == 500 else '南都 6-GFM-155HR'
    # 电池组数
    sheet2.range('G41').value = 2 if powerclass == 300 else 3 if powerclass == 500 else 4
    # 系统类型
    sheet2.range('G35').value = '单机' if powerclass == 300 else '分散并机'
    # 外部维修旁路
    sheet2.range('G36').value = '有-3P'
    # 内部开关配置
    sheet2.range('R33').value = '主路输入、旁路、输出' if powerclass == 600 else '主路输入、旁路、输出、维修旁路'
    # 并机输出开关
    sheet2.range('R44').value = '无'
    # 时间
    sheet2.range('E462').value = doctime
    sheet2.range('Q462').value = doctime
    # 并机/本机编码/产品条码
    # 单独处理

    # Mains
    Mains = data['Mains']
    # 定义一个二维数组存储数据
    Mains_handle = [[0 for y in range(3)] for x in range(3)]
    Mains_index = 0
    # 对数据进行处理
    for k, l in Mains.items():
        for i, v in enumerate(l):
            if v[-2:] == 'Hz':
                Mains_handle[Mains_index][i] = v[:-3].strip()
            elif v[-1:] == 'V':
                Mains_handle[Mains_index][i] = round(float(v[:-1]) * math.sqrt(3), 2)
            else:
                Mains_handle[Mains_index][i] = v[:-1].strip()
        Mains_index += 1
    # 将数据渲染至目标位置 - 纵向
    sheet2.range('G167').options(transpose=True).value = Mains_handle[0]
    sheet2.range('K167').options(transpose=True).value = Mains_handle[1]
    sheet2.range('O167').options(transpose=True).value = Mains_handle[2]
    # sheet2.range('G167:R169').api.HorizontalAlignment = -4131

    # Reserve
    Reserve = data['Reserve']
    Reserve_handle = [[0 for y in range(3)] for x in range(3)]
    Reserve_index = 0
    for k, l in Reserve.items():
        # 清空二维数组初始化元素
        Reserve_handle[Reserve_index].clear()
        for i, v in enumerate(l):
            if v[-1] == 'V':
                Reserve_handle[Reserve_index].append(v[:-1].strip())
            elif v[-2:] == 'Hz':
                Reserve_handle[Reserve_index].append(v[:-2].strip())
        Reserve_index += 1
    sheet2.range('G178').options(transpose=True).value = Reserve_handle[0]
    sheet2.range('K178').options(transpose=True).value = Reserve_handle[1]
    sheet2.range('O178').options(transpose=True).value = Reserve_handle[2]
    # sheet2.range('G178:R179').api.HorizontalAlignment = -4131

    # output
    Output = data['Output']
    Output_handle = [[0 for y in range(9)] for x in range(3)]
    Output_index = 0
    for k, l in Output.items():
        for i, v in enumerate(l):
            # 两个行被清空了 ...
            Output_handle[Output_index][2] = ''
            Output_handle[Output_index][5] = ''
            if v[-1] == 'V':
                Output_handle[Output_index][0] = round(float(v[:-1].strip()) * math.sqrt(3), 2)
                Output_handle[Output_index][1] = v[:-1].strip()
            elif v[-2:] == 'Hz':
                Output_handle[Output_index][3] = v[:-2].strip()
            elif v[-1] == 'A':
                if v[-3:] == 'kVA':
                    Output_handle[Output_index][6] = v[:-3].strip()
                else:
                    Output_handle[Output_index][4] = v[:-1].strip()
            elif v[-2:] == 'PF':
                Output_handle[Output_index][7] = abs(float('0 PF'[:-2].strip()))
            else:
                Output_handle[Output_index][8] = float(l[3][:-3].strip()) * 3 / powerclass
        Output_index += 1
    sheet2.range('G263').options(transpose=True).value = Output_handle[0]
    sheet2.range('K263').options(transpose=True).value = Output_handle[1]
    sheet2.range('O263').options(transpose=True).value = Output_handle[2]
    # sheet2.range('G263:R269').api.HorizontalAlignment = -4131

    # 保存
    wb.save()
    wb.close()
    app.quit()
    time.sleep(0.2)

# 创建 excel 表格并写入 html 解析的数据
def createXlsx(data, filename):
    app = xw.App(visible=False,add_book=False)
    app.display_alerts = False
    app.screen_updating = False
    wb = app.books.add()
    sheet = wb.sheets['sheet1']
    # 写入表格数据
    count = 1
    for k, v in data.items():
        '''
        单元格区域合并
        sheet2.range('A1:B1').merge()
        单元格区域清空
        sheet2.range('A1:D3').clear()
        设置字体为粗体
        sheet2.range('A1').api.Font.Bold = True
        水平居中
        sheet2.range('A1').api.HorizontalAlignment = -4108
        单元格格式：http://www.dszhp.com/xlwings-format.html
        '''
        sheet.range('A' + str(count) + ':G' + str(count)).merge()
        sheet.range('A' + str(count)).value = k
        sheet.range('A' + str(count)).api.Font.Bold = True
        sheet.range('A' + str(count)).api.Font.Size = 16 
        count += 1
        if not isinstance(v, list):
            for i in v:
                sheet.range('A' + str(count)).value = i
                sheet.range('A' + str(count)).api.Font.Bold = True
                sheet.range('B' + str(count)).value = v[i]
                count += 1
        else:
            sheet.range('B' + str(count)).value = v
    # 整体居中
    sheet.range('A1:G' + str(count)).api.HorizontalAlignment = -4108

    if not os.path.isdir(output_path[2:-1]):
        os.mkdir(r'' + output_path[2:-1])
    wb.save(output_path + filename + '.xlsx')
    wb.close()
    app.quit()
    time.sleep(0.2)

# 遍历每个 html
def parseHtml(file, filename):
    # 得到整个文档 dom
    doc = BeautifulSoup(open(file), from_encoding='utf-8', features='html.parser')

    # 得到时间
    time_dom = doc.find('h3')
    time_compile = re.compile('\d{4}-\d{2}-\d{2}')
    doctime = re.search(time_compile, str(time_dom))[0].replace('-', '/', 2)

    # 得到功率等级
    power_class_dom = doc.find('table')
    power_class_compile = re.compile(r'<td>(\d+)kVA')
    power_class = re.search(power_class_compile, str(power_class_dom))[1]

    # 得到设备编码 sn
    sn_number_dom = doc.find('h1')
    sn_number_compile = re.compile('\((.+)\)')
    sn_number = re.search(sn_number_compile, str(sn_number_dom))[1]

    # 得到对应 meatures 下的表格节点
    measures = doc.find('h3', string='Measures')
    table = measures.find_parent().find_next_sibling()

    # 获得表头字段
    table_head = table.find('thead').find_all('td')
    for td in table_head:
        if td.string != None:
            table_head_list.append(td.string)

    # 获得表身数据 并整理为一个对象
    table_body = table.find('thead').find_next_sibling().contents
    table_body_list = {}
    table_body_item_flag = 0
    for x in table_body:
        if re.search(re.compile('table'), str(x)):
            item = x.find_all('td')
            item_transform = {}
            # 当前 L1, L2, L3 的计数
            curr_L = []
            L_index = 1
            L_length = 3
            # 当前遍历 td 的长度
            item_length = len(item)
            # 当前遍历 td 内元素的计数
            el_length = item_length // L_length
            el_last_length = item_length - el_length * 2
            el_index = 1
            for v in item:
                v = v.string
                curr_L.append(v)
                # # 对 battery 特殊处理
                if table_body_item_flag == 3:
                    item_transform = curr_L
                elif (el_index == el_length and L_index != 3) or (L_index == 3 and el_index == el_last_length):
                    item_transform['L' + str(L_index)] = curr_L
                    curr_L = []
                    el_index = 0
                    L_index += 1
                el_index += 1
            # 数组拍平 -----
            # item_transform = curr_L
            # -------------
            table_body_list[table_head_list[table_body_item_flag]] = item_transform
            table_body_item_flag += 1
            if table_body_item_flag >= 4:
                break

    # 对表格进行补全
    export_excel = table_body_list.copy()
    for k, v in export_excel.items():
        if not isinstance(v, list):
            v['L1'].append('50 Hz')
            v['L2'].append('50 Hz')
    # 记录当前提取内容
    with open('result.txt', 'wb') as file:
        file.write(json.dumps(table_body_list, indent=4, ensure_ascii=False, sort_keys=True).encode())
    # 解析后的内容创建 excel 记录
    # createXlsx(export_excel, filename)
    # 若解析的文档存在与之对应的待编辑 xlsb 文件
    if filename in xlsb_list:
        editMatchedXlsb(table_body_list, filename, doctime, int(power_class), sn_number)
    else:
        print('提供的 excel 文档中不存在 [' + filename + ']')

# 获取统计数据 stat
def handleStat():
    # 开始新的遍历获得 stat_list
    finished_doc = 1
    for v in xlsb_list:
        app = xw.App(visible=False,add_book=False)
        #不显示Excel消息框
        app.display_alerts = False
        #关闭屏幕更新,可加快宏的执行速度
        app.screen_updating = False

        wb = app.books.open(xlsb_path + v + '.xlsb')
        sheet = wb.sheets['巡检报告']

        # 并机数/本机编号
        sheet.range('R43').value = stat_list[v][2]
        sheet.range('G43').value = stat_list[v][3]
        # 并机条码
        snnumber_range = ['G55', 'R55', 'G56']
        for i, s in enumerate(stat_list[v][1]):
            sheet.range(snnumber_range[i]).value = s
        # 并机检测结果
        sheet.range('T379').value = '正常'
        sheet.range('T381').value = '正常'
        sheet.range('T382').value = '未测试'

        # 导出 pdf
        # wb.to_pdf(r'' + xlsb_path + v + '.pdf', ['封面', '巡检报告'])
        # 退出
        wb.save()
        wb.close()
        app.quit()
        time.sleep(0.2)
        print('[' + v[0] + '] 数据已整理，当前已有' + str(finished_doc) + '个完成')
        finished_doc += 1

def main():
    # 判断目标文件夹是否存在
    if not os.path.isdir(xlsb_path[2:-1]):
        os.mkdir(r'' + xlsb_path[2:-1])
        print('请将代编辑的 excel 文档移入文件夹: ' + xlsb_path[2:-1])
        return
    if not os.path.isdir(html_path[2:-1]):
        os.mkdir(r'' + html_path[2:-1])
        print('请将代编辑的 html 文档移入文件夹: ' + html_path[2:-1])
        return

    # 获取代编辑文件，收集名字为数组，写入后从数组删除
    xlsb_files_list = os.listdir(xlsb_path)
    for i, v in enumerate(xlsb_files_list):
        a, b = os.path.splitext(v)
        if b == ('.xlsb' or '.xlsx'):
            xlsb_list.append(a)

    # 排序处理，之后在获取并机数据时需要按序处理
    xlsb_list.sort()

    # 开启监听
    listener = MsgBoxListener(2)
    listener.start()

    # 抓取 html 数据并写入目标文件
    html_list = os.listdir(html_path)
    finished_doc = 1
    for i in html_list:
        a, b = os.path.splitext(i)
        if b == '.html':
            parseHtml(html_path + i, a)
            print('[' + a + '] 文件已解析，当前已有' + str(finished_doc) + '个完成')
            finished_doc += 1

    # 处理统计文件 statistics-doc
    print("\n")
    print('－－－－－－－－－－－－－－－－－－－－－－－－')
    print('进行数据整合，请等待...')
    handleStat()
    print('处理完毕！')
    listener.stop()

if __name__ == '__main__':
    main()
```