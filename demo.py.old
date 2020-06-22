#!/usr/bin/env python
# -*- coding:utf-8 -*-

import pandas as pd
import numpy as np
import os
import json
import tkinter.messagebox
from tkinter import *
from PIL import Image, ImageTk
import matplotlib.pyplot as plt
from matplotlib.backends.backend_agg import FigureCanvasAgg
plt.rcParams['font.sans-serif'] = ['SimHei', 'Times New Roman']


def pie_chart(name, labels, values):
    '''
    根据年度或者月度收支汇总和类别，画饼图
    :param name:饼图名字
    :param labels:收支明细类别
    :param values:收支明细
    :return: image->np.array
    '''
    def my_label(pct, allvals):
        absolute = pct / 100. * np.sum(allvals)
        return "{:.1f}%\n{:.1f}元".format(pct, absolute)

    explode = np.zeros_like(values)
    explode[0] = .1

    plt.pie(values, explode=explode,
            labels=labels, autopct=lambda x: my_label(x, values),
            startangle=180, shadow=True,
            colors=['c', 'r', 'gray', 'g', 'y'])

    # 标题
    plt.title(name)
    canvas = FigureCanvasAgg(plt.gcf())
    canvas.draw()
    plt.close()
    img = np.array(canvas.renderer.buffer_rgba())
    return img


def histogram(year, months, earns, pays):
    '''
    根据年度汇总绘制每个月收支的直方图
    :param year:年度
    :param months:月份数
    :param earns:每月收入汇总
    :param pays:每月支出汇总
    :return:image->np.array
    '''
    plt.bar(x=months, height=earns, label='收入', color='steelblue', alpha=0.8)
    plt.bar(x=months, height=pays, label='支出', color='indianred', alpha=0.8)
    # 在柱状图上显示具体数值, ha参数控制水平对齐方式, va控制垂直对齐方式
    for x, y in enumerate(earns):
        plt.text(x, y, '%s元' % y, ha='center', va='bottom')
    for x, y in enumerate(pays):
        plt.text(x, y, '%s元' % y, ha='center', va='top')
    # # 设置标题
    plt.title("{}年各月收入支出柱状图".format(year))
    # 为两条坐标轴设置名称
    plt.xlabel("月份")
    plt.ylabel("收支金额")
    # 显示图例
    plt.legend()
    canvas = FigureCanvasAgg(plt.gcf())
    canvas.draw()
    plt.close()
    img = np.array(canvas.renderer.buffer_rgba())
    return img


class PersonalFinancialManager(object):
    '''
    个人收支管理系统后台
    包括收支类别定义、记账、月度汇总、年度汇总功能。
    '''
    def __init__(self):
        self.file = './income_expenditure.csv'
        self.define_file = './define_ab.json'
        if os.path.exists(self.define_file):
            with open(self.define_file, 'r', encoding='utf-8') as f:
                load_dict = json.load(f)
                self.a = load_dict['A']
                self.b = load_dict['B']
        else:
            self.a = {}
            self.b = {}
        if os.path.exists(self.file):
            self.data = pd.read_csv(self.file, encoding="utf-8",
                                    dtype={'日期': int,
                                           '金额': float})
        else:
            self.data = pd.DataFrame(columns=['收/支', '类别', '日期', '金额', '备注'])

    def define(self, flag, key, name):
        '''
        定义收支类别
        :param flag: 收/支
        :param key: 收支类别编码
        :param name: 收支类别名字
        '''
        if flag == 0:
            self.a[key] = name
        elif flag == 1:
            self.b[key] = name

    def save_define(self):
        '''
        将收支类别写入文件
        :return:
        '''
        define_dict = {'A': self.a, 'B': self.b}
        try:
            jsObj = json.dumps(define_dict)
            fileObject = open(self.define_file, 'w', encoding='utf-8')
            fileObject.write(jsObj)
            fileObject.close()
            return True
        except:
            return False

    def write(self, inputs):
        '''
        记账
        :param inputs: 记账信息
        :return:  ret： 记账是否成功
                  tip： 提示信息
        '''
        assert isinstance(inputs, str)
        tip1 = '请按照\'类别编码，发生日期，金额，备注\'的格式输入，中间用英文逗号隔开！例如：a1,2020-6-1,20.5,冰淇淋'
        tip2 = '请输入已定义的正确类别编码！例如：a1'
        tip3 = '请按照\'年-月-日\'的格式输入！例如：2020-6-1'
        tip4 = '请输入正确的数字金额! 例如：20.5'
        inputs = inputs.split(',')
        if len(inputs) != 4:
            return False, tip1

        if inputs[0] in self.a.keys():
            inputs.insert(0, '收入')
            inputs[1] = self.a[inputs[1]]
        elif inputs[0] in self.b.keys():
            inputs.insert(0, '支出')
            inputs[1] = self.b[inputs[1]]
        else:
            return False, tip2

        date_list = inputs[2].split('-')
        if len(date_list) != 3:
            return False, tip3
        try:
            date_list = list(map(lambda x: int(x), date_list))
            if date_list[0] < 100:
                date_list[0] += 2000
            date = '{:04d}{:02d}{:02d}'.format(date_list[0], date_list[1], date_list[2])
            inputs[2] = int(date)
        except ValueError:
            return False, tip3
        try:
            inputs[3] = float(inputs[3])
        except ValueError:
            return False, tip4
        self.data.loc[self.data.shape[0], :] = inputs
        return True, '输入完成！'

    def delete(self,index):
        '''
        根据索引删除某些记录
        :param index:
        :return:
        '''
        tip = '输入要删除的索引，以英文逗号隔开！'
        try:
            index_list = list(map(lambda x: int(x), index.split(',')))
        except:
            return False,tip
        self.data.drop(index_list, inplace=True, axis=0)
        return True,'OK'

    def save(self):
        '''
        写入当前账本信息
        :return:  成功/失败
        '''
        self.data.sort_values(by=['日期', '类别'], ascending=[True, True], inplace=True)
        try:
            self.data.to_csv(self.file, index=False)
            return True
        except:
            return False

    def summary(self, month):
        '''
        月度汇总
        :param month: 月份
        :return: ret， 是否成功
                month_summary, 月度汇总
                month_data, 月度明细
                pay_chart, 支出的饼图
                earn_chart， 收入的饼图
        '''
        month_raw = month
        tip = '请按照\'年-月\'的格式输入月份！例如：2020-6'
        tip1 = '该月没有记录'
        month_list = month.split('-')
        if len(month_list) != 2:
            return False, tip,0,0,0
        try:
            month_list = list(map(lambda x: int(x), month_list))
            if month_list[0] < 100:
                month_list[0] += 2000
            month = '{:04d}{:02d}'.format(month_list[0], month_list[1])
            month = int(month)
        except ValueError:
            return False, tip,0,0,0
        month_data = self.data[self.data.日期 // 100 == month]
        month_data = month_data.reset_index(drop=True)
        if len(month_data)==0:
            return False, tip1, 0, 0, 0
        month_summary = month_data.pivot_table(['金额'], ['收/支', '类别'], aggfunc='sum')
        pay = month_summary.金额.支出
        earn = month_summary.金额.收入
        pay_chart = pie_chart('{}月支出汇总'.format(month_raw), pay.index, pay.values)
        earn_chart = pie_chart('{}月收入汇总'.format(month_raw), earn.index, earn.values)
        return True, month_summary, month_data, pay_chart, earn_chart

    def yearly_summary(self, year):
        '''
        年度汇总统计
        :param year: 年份
        :return: ret， 是否成功
                 pay_chart, 支出的饼图
                 earn_chart, 收入的饼图
                 months_his, 每月收支的直方图
        '''
        year_raw = year
        tip1 = '请输入正确的年份数字！'
        tip2 = '该年没有数据！'
        tip3 = '该年没有数据！'
        try:
            year = int(year)
        except:
            return False, tip1,0,0
        if year < 100: year += 2000
        if year > 3000: return False, tip1
        year_data = self.data[self.data.日期 // 10000 == year]

        # 饼图
        try:
            year_summary = year_data.pivot_table(['金额'], ['收/支', '类别'], aggfunc='sum')
            pay = year_summary.金额.支出
            earn = year_summary.金额.收入
            pay_chart = pie_chart('{}年支出汇总'.format(year_raw), pay.index, pay.values)
            earn_chart = pie_chart('{}年收入汇总'.format(year_raw), earn.index, earn.values)
        except:
            return False, tip2,0,0

        # 直方图
        try:
            monlist = []
            for i in set(year_data.日期 // 100):
                month_data = year_data[year_data.日期 // 100 == i]
                month_summary = month_data.pivot_table(['金额'], ['收/支'], aggfunc='sum')
                try:
                    earn = month_summary.loc['收入'].金额
                except:
                    earn = 0
                try:
                    pay = month_summary.loc['支出'].金额
                except:
                    pay = 0
                monlist.append([str(i)[-2:], earn, pay])
            months, earns, pays = zip(*monlist)
            months_his = histogram(year, months, earns, pays)
        except:
            return False, tip3,0,0
        return True, pay_chart, earn_chart, months_his


class homepage(object):
    '''
    主页，包含 “定义收支类别”,“记账”，“月度汇总”，“年度查询汇总” 4个部分
    '''

    def __init__(self):
        self.root = Tk()
        self.root.title('个人收支管理系统')
        self.root.geometry("500x500+750+300")
        Button(self.root, text="定义收支类别", command=self.go_define_page,height=5,width=40,bd=4).pack()
        Button(self.root, text="记一笔账", command=self.go_record_page,height=5,width=40,bd=4).pack()
        Button(self.root, text="月度查询汇总", command=self.go_month_summary_page,height=5,width=40,bd=4).pack()
        Button(self.root, text="年度查询汇总", command=self.go_year_summary_page,height=5,width=40,bd=4).pack()
        self.root.mainloop()

    def go_define_page(self):
        """
        跳转函数
        """
        self.root.destroy()
        define_page()

    def go_record_page(self):
        self.root.destroy()
        record_page()

    def go_month_summary_page(self):
        self.root.destroy()
        month_summary_page()

    def go_year_summary_page(self):
        self.root.destroy()
        year_summary_page()




class define_page(object):
    '''
    定义收入和支出的细分类别，显示当前类别情况
    :return:
    '''
    def __init__(self):
        self.pfm = PersonalFinancialManager()
        self.a = {}
        self.b = {}
        self.root = Tk()
        self.root.title('定义收支类别')
        self.root.geometry("500x500+750+300")
        Label(self.root, text="定义收入类别").grid(row=0, column=0)
        for i in range(1,7):
            Label(self.root, text="a%d"%i).grid(row=i, column=0)
            self.a['a%d'%i] = Entry(self.root)
            self.a['a%d'%i].grid(row=i, column=1, columnspan=2)
            try:
                present_value = self.pfm.a['a%d'%i]
                Label(self.root, text="当前为：%s"%present_value).grid(row=i, column=3, columnspan=3)
            except:
                continue
        Label(self.root, text="定义支出类别").grid(row=7, column=0)
        for i in range(8,14):
            Label(self.root, text="b%d"%(i-7)).grid(row=i, column=0)
            self.b['b%d'%(i-7)] = Entry(self.root)
            self.b['b%d' % (i - 7)].grid(row=i, column=1, columnspan=2)
            try:
                present_value = self.pfm.b['b%d'%(i-7)]
                Label(self.root, text="当前为：%s"%present_value).grid(row=i, column=3, columnspan=3)
            except:
                continue

        Button(self.root, text="确定", command=self.define).grid(row=15, column=0,rowspan=15)
        Button(self.root, text="返回", command=self.go_homepage).grid(row=15, column=3, rowspan=15)
        self.root.mainloop()

    def go_homepage(self):
        self.root.destroy()
        homepage()

    def tip1(self):
        tkinter.messagebox.showerror(title='错误：', message='必须定义3至6个收入或者支出类别！')

    def tip2(self):
        tkinter.messagebox.showerror(title='错误：', message='保存文件失败！')

    def define(self):
        for i in range(1,7):
            src = self.a['a%d'%i].get().strip()
            if src:
                self.pfm.define(0,'a%d'%i,src)
        for i in range(1,7):
            src = self.b['b%d'%i].get().strip()
            if src:
                self.pfm.define(1,'b%d'%i,src)
        if 3<=len(self.pfm.a.keys())<=6 and 3<=len(self.pfm.b.keys())<=6:
            ret = self.pfm.save_define()
            if ret: self.go_homepage()
            else:
                self.tip2()
        else:
            self.tip1()

class record_page(object):
    '''
    记账页面   再记一笔 和 完成
    :return:
    '''
    def __init__(self):
        self.pfm = PersonalFinancialManager()
        self.root = Tk()
        self.root.title('记一笔账')
        self.root.geometry("500x500+750+300")
        self.sb = Scrollbar(self.root)
        self.sb.pack(side=RIGHT, fill=Y)
        Label(self.root,
              text="输入收支明细, 例如：b1,2020-6-1,20.5,冰淇淋 \n输入流水的索引(英文逗号隔开)点击删除可以删除多条记录"
              ).pack()
        self.input_detail = Text(self.root, width=40, height=1)
        self.input_detail.pack()
        Button(self.root, text="写入", command=self.reopen).pack()
        Button(self.root, text="删除", command=self.delete).pack()
        Button(self.root, text="返回", command=self.go_homepage).pack()
        Label(self.root, text="流水显示").pack()
        self.text = Text(self.root, width=80, height=40,yscrollcommand=self.sb.set)
        self.text.insert(1.0,self.pfm.data)
        self.text.pack()
        self.sb.config(command=self.text.yview)

        self.root.mainloop()

    def go_homepage(self):
        self.root.destroy()
        homepage()

    def reopen(self):
        self.record()
        self.root.destroy()
        self.__init__()

    def delete(self):
        src = self.input_detail.get(1.0, END).strip()
        ret,tip = self.pfm.delete(src)
        if ret:
            self.pfm.save()
            tkinter.messagebox.showinfo(title=None, message=tip)
        else:
            tkinter.messagebox.showerror(title='错误：', message=tip)
        self.root.destroy()
        self.__init__()

    def record(self):
        src = self.input_detail.get(1.0,END).strip()
        ret,tip = self.pfm.write(src)
        if ret:
            self.pfm.save()
            tkinter.messagebox.showinfo(title=None, message=tip)
        else:
            tkinter.messagebox.showerror(title='错误：', message=tip)

class month_summary_page(object):
    '''
    月度汇总页面  输入月份，显示汇总和饼图
    :return:
    '''
    def __init__(self):
        self.pfm = PersonalFinancialManager()
        self.root = Tk()
        self.root.title('月度汇总查询')
        self.root.geometry("1800x1000+100+20")
        self.summary_tip = ''
        self.detail_tip = ''
        Label(self.root, text="输入要查询的月份，例如：2020-6").grid(row=0, column=0)
        self.input_detail = Entry(self.root)
        self.input_detail.grid(row=1, column=0)
        Button(self.root, text="查询", command=self.summary).grid(row=2, column=0)
        Button(self.root, text="重新查询", command=self.reopen).grid(row=2, column=1, columnspan=1)
        Button(self.root, text="返回", command=self.go_homepage).grid(row=3, column=1)
        Label(self.root, text="月度汇总").grid(row=4, column=0)
        self.result_data_Text = Text(self.root, width=50, height=30)  # 处理结果展示
        self.result_data_Text.grid(row=4, column=0,rowspan=10)
        Button(self.root, text="查询明细", command=self.detail).grid(row=3, column=0)
        Label(self.root, text="明细").grid(row=40, column=0)
        self.detail_data_Text = Text(self.root, width=50, height=40)  # 处理结果展示
        self.detail_data_Text.grid(row=50, column=0, rowspan=10)
        self.root.mainloop()

    def reopen(self):
        self.root.destroy()
        self.__init__()

    def summary(self):
        month = self.input_detail.get().strip()
        ret,month_summary, month_data, pay_chart, earn_chart = self.pfm.summary(month)
        if ret:
            self.summary_tip = str(month_summary)
            self.detail_tip = str(month_data)
            self.img_earn = ImageTk.PhotoImage(Image.fromarray(earn_chart))
            label_img = Label(self.root, image=self.img_earn)
            label_img.grid(row=5, column=50, columnspan=1)
            self.img_pay = ImageTk.PhotoImage(Image.fromarray(pay_chart))
            label_img = Label(self.root, image=self.img_pay)
            label_img.grid(row=5, column=60, columnspan=1)
            self.result_data_Text.delete(1.0, END)
            self.result_data_Text.insert(1.0, self.summary_tip)
        else:
            tip = month_summary
            tkinter.messagebox.showerror(title=None, message=tip)

    def detail(self):
        self.detail_data_Text.delete(1.0, END)
        self.detail_data_Text.insert(1.0, self.detail_tip)

    def go_homepage(self):
        self.root.destroy()
        homepage()

class year_summary_page(object):
    '''
    年度汇总页面  输入年份，显示柱状图和饼图
    :return:
    '''
    def __init__(self):
        self.pfm = PersonalFinancialManager()
        self.root = Tk()
        self.root.title('年度汇总查询')
        self.root.geometry("1800x1080+100+0")
        self.summary_tip = ''
        Label(self.root, text="输入要查询的年度，例如：2020").place(relx=0,rely=0)
        self.input_detail = Entry(self.root)
        self.input_detail.place(x=10,y=20)
        Button(self.root, text="查询", command=self.summary).place(x=10,y=40)
        Button(self.root, text="重新查询", command=self.reopen).place(x=60,y=40)
        Button(self.root, text="返回", command=self.go_homepage).place(x=130,y=40)
        self.root.mainloop()

    def summary(self):
        month = self.input_detail.get().strip()
        ret,pay_chart, earn_chart,his = self.pfm.yearly_summary(month)
        if ret:
            self.img_earn = ImageTk.PhotoImage(Image.fromarray(earn_chart))
            label_img = Label(self.root, image=self.img_earn)
            label_img.place(relx=.1,rely=0)
            self.img_pay = ImageTk.PhotoImage(Image.fromarray(pay_chart))
            label_img = Label(self.root, image=self.img_pay)
            label_img.place(relx=.5,rely=0)
            self.img_his = ImageTk.PhotoImage(Image.fromarray(his))
            label_img = Label(self.root, image=self.img_his)
            label_img.place(relx=.1,rely=.5)
        else:
            tip = pay_chart
            tkinter.messagebox.showerror(title=None, message=tip)

    def reopen(self):
        self.root.destroy()
        self.__init__()

    def go_homepage(self):
        self.root.destroy()
        homepage()

if __name__ == '__main__':
    homepage()


