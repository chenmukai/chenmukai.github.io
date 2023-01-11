---
title: script
date: 2022-07-15 23:00:26
tags:
  - python
categories:
  - python
keywords: "python"
---

## 1.子图切割

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

"""
@function: 根据txt标签文件，对图片进行切割子图
"""

import os
import argparse
from datetime import datetime
from tqdm import tqdm
import cv2 as cv
import numpy as np


parser = argparse.ArgumentParser(description='crop imgs')
parser.add_argument('--roots', type=str, help='操作源文件路径', default=r'')
parser.add_argument('--save_path', type=str, help='保存路径', default=r'')
parser.add_argument('--extend_pix', type=int, help='外扩像素点', default=0)
parser.add_argument('--coor_tran', type=bool, help='进行坐标转换', default=True)
args = parser.parse_args()
data_path = args.roots
crop_path = args.save_path
extend_pixel = args.extend_pix

if not os.path.exists(crop_path):
    os.makedirs(crop_path)

def get_files(roots):
    imgs = []
    postfix = ('.jpg', 'JPG', 'jpeg', '.JPEG', '.png', '.PNG', '.bmp', '.BMP', '.gif', '.GIF')
    for root, dirs, files in os.walk(roots):
        for file in files:
            if file.endswith(postfix):
                imgs.append(os.path.join(root, file))
    return imgs

def coordinate_transform(img, roi):
    height,width,_ = img.shape
    cls_name = roi[0]
    x1 = int((float(roi[1]) - 1 / 2 * float(roi[3])) * width)
    y1 = int((float(roi[2]) - 1 / 2 * float(roi[4])) * height)
    x2 = int((float(roi[1]) + 1 / 2 * float(roi[3])) * width)
    y2 = int((float(roi[2]) + 1 / 2 * float(roi[4])) * height)
    coor = [cls_name, x1, y1, x2, y2]
    return coor

def crop_image(img_path, roi, img_name, idx):
    image = cv.imdecode(np.fromfile(img_path, dtype=np.uint8), -1)
    h,w,_ = image.shape
    if args.coor_tran:
        roi = coordinate_transform(image, roi)
    
    x1 = 0 if roi[1] - extend_pixel < 0 else roi[1] - extend_pixel
    y1 = 0 if roi[2] - extend_pixel < 0 else roi[2] - extend_pixel
    x2 = w if roi[3] + extend_pixel > w else roi[3] + extend_pixel
    y2 = h if roi[4] + extend_pixel > h else roi[4] + extend_pixel
    img_crop = image[y1:y2, x1:x2]
    
    if roi[0].endswith(('*')):
        roi[0] = roi[0].split('*')[0] + '_'

    classDir = os.path.split(crop_path + img_path.replace(data_path, ''))[0] + '\\' + str(roi[0])

    if not os.path.exists(classDir):
        os.makedirs(classDir)
    dst_path = os.path.join(classDir, img_name + '_' + str(idx) + '.jpg')
    cv.imencode('.jpg', img_crop)[1].tofile(dst_path)


def run_main(imgs):
    
    with tqdm(range(len(imgs))) as pbar:
        for img_path in imgs:
            flag = False
            try:
                img_name = img_path.split('\\')[-1].split('.')[0]
                txt_path = img_path.replace('.jpg', '.txt')

                with open(txt_path, "r", encoding='utf-8') as f:
                    data = f.readlines()
                    idx = 0

                    for tmp in data:
                        if tmp.split(' ')[0] == '坐标信息数据为空\n':
                            flag = True
                            break
                        cls_name, x, y, w, h = tmp.split(' ')[0:5]
                        if '\n' in h:
                            h = h.split('\n')[0]
                        roi = [cls_name, x, y, w, h]
                        crop_image(img_path, roi, img_name, idx)
                        idx += 1
                if flag:
                    os.remove(img_path)
                    os.remove(txt_path)
                    continue

            except:
                print('[image error] -> {}'.format(img_path))
            pbar.update(1)

if __name__ == '__main__':
    start = datetime.now()
    print('start crop')
    imgs = get_files(data_path)
    run_main(imgs)
    print('finish！')
    print('total times: {}'.format(datetime.now() - start))
```



## 2.标签相关操作

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

'''
@function: 遍历txt文件统计、(批量或指定)修改、删除、标签类别信息
'''

import imp
import os
import multiprocessing as mp
from datetime import datetime
import shutil
from tqdm import tqdm


def cost_time(func):
    def wrapper(*arg, **kwarg):
        start_time = datetime.now()
        execute_func = func(*arg,**kwarg)
        end_time = datetime.now()
        print("Finish! 执行【{}】耗时{} ".format(func.__name__,(end_time - start_time)))
        print("---------------------------------------")
        return execute_func
    return wrapper

def get_files(roots):
    txt_list = []
    img_list = []
    img_postfix = ('.jpg', 'JPG', 'jpeg', '.JPEG', '.png', '.PNG', '.bmp', '.BMP', '.gif', '.GIF')
    for root, dirs, files in os.walk(roots):
        if len(files) == 0:
            continue
        for file in files:
            if file.endswith('txt'):
                txt_list.append(os.path.join(root, file))
            if file.endswith(img_postfix):
                img_list.append(os.path.join(root, file))
    return txt_list, img_list

@cost_time
def modify_label(files, obj_label_id, modify_label_id):

    with tqdm(range(len(files))) as pbar:
        for file in files:
            with open(file, "r") as f1, open("%s.bak"%file, "w") as f2:
                try:
                    data = f1.readlines()
                    for tmp in data:
                        tt = tmp.split(" ")

                        if modify_label_id == None: 
                            tt[0] = obj_label_id  # 全部修改成obj_label_id
                        elif tt[0]==modify_label_id:
                            tt[0] = obj_label_id

                        tt[4] = tt[4].split("\n")[0]
                        cnt = 0
                        for temp in tt:
                            if len(temp) > 8:
                                f2.write(temp[:8])
                            else:
                                f2.write(temp)
                            if cnt <= 3:
                                f2.write(" ")
                            else:
                                pass
                            cnt += 1
                        f2.write("\n")
                except:
                    print('empty txt file: {}'.format(file))
                pbar.update(1)
            os.remove(file)
            os.rename("%s.bak"%file, file)

@cost_time
def static_label(files, targetIds=['0','1','2','3']):
    label_dic = {}
    with tqdm(range(len(files))) as pbar:
        for file in files:
            imgPath = file.replace('.txt', '.jpg')
            imgName = os.path.split(imgPath)[-1]
            txtName = os.path.split(file)[-1]
            removeFlag = False
            with open(file, "r") as f1:
                try:
                    data = f1.readlines()
                    for tmp in data:
                        tt = tmp.split(" ")
                        if tt[0] not in targetIds:
                            removeFlag = True
                        # 统计各个标签的数量
                        if tt[0] in sorted(label_dic):
                            label_dic[tt[0]] += 1
                        else:
                            label_dic[tt[0]] = 1
                except:
                    print('empty txt file: {}'.format(file))
            if removeFlag:
                os.remove(imgPath)
                os.remove(file)
            pbar.update(1)
    print(label_dic)

@cost_time
def remove_label_line(files, label_id, save=False):

    for file in files:
        if save:
            lines = (i for i in open(file,'r') if label_id in i.split(' '))
        else:
            lines = (i for i in open(file,'r') if label_id not in i.split(' '))
        f = open("%s.bak"%file, "w", encoding='utf-8')
        f.writelines(lines)
        f.close()
        os.remove(file)
        os.rename("%s.bak"%file, file)
        

@cost_time
def map_file(imgs, txts, save_path):
    if save_path and not os.path.exists(save_path):
        os.mkdir(save_path)

    img_len = len(imgs)
    txt_len = len(txts)

    if img_len == txt_len:
        return
    elif img_len > txt_len:
        move_file(imgs, txts, '.jpg', '.txt', save_path)
        print("去除多余图片 {} 张".format(img_len-txt_len))
    else:
        move_file(txts, imgs, '.txt', '.jpg', save_path)
        print("去除多余标签 {} 个".format(txt_len-img_len))

def move_file(src_list, target_list, src_postfix, target_postfix, save_path):
    for file in src_list:
        fileName = os.path.split(file)[-1]
        judgeFile = file.replace(src_postfix, target_postfix)
        if judgeFile not in target_list:
            if not save_path:
                os.remove(file)
            else:
                shutil.move(file, os.path.join(save_path, fileName))


@cost_time
def copy_label_file(files, ids, save_path):
    if not os.path.exists(save_path):
        os.makedirs(save_path)
    with tqdm(range(len(files))) as pbar:
        count = 0
        for file in files:
            imgPath = file.replace('.txt', '.jpg')
            imgName = os.path.split(imgPath)[-1]
            txtName = os.path.split(file)[-1]
            with open(file, "r") as f1:
                try:
                    data = f1.readlines()
                    for tmp in data:
                        tt = tmp.split(" ")
                        if tt[0] in ids:
                            shutil.copy(imgPath, os.path.join(save_path,imgName))
                            shutil.copy(file, os.path.join(save_path, txtName))
                            # shutil.move(imgPath, os.path.join(save_path,imgName))
                            # shutil.move(file, os.path.join(save_path, txtName))
                            count+=1
                            break
                except:
                    print('empty txt file: {}'.format(file))
                pbar.update(1)
        print("拷贝目标类别[{}]:{}张".format(id,count))




if __name__=='__main__':

    roots = input("请输入操作源文件路径（初始）：")
    txt_list, img_list = get_files(roots)
    
    methods = {
        'staticLabel': 1,
        'modifyLabel': 2,
        'removeLabel': 3,
        'mapFile': 4,
        'copyLableFile': 5,
        'changeRoot': 6,
        'exit':0
    }
    while True:
        for key, value in methods.items():
            print('\t功能【'+ key + '】请选择：' + str(value))
        method_id = int(input("请输入要执行的方法："))
        mode = list(methods.keys())[list(methods.values()).index(method_id)]
        
        if mode == 'staticLabel':
            print("执行【统计标签各类别数】！")
            static_label(txt_list)

        elif mode == 'modifyLabel':
            print("执行【修改标签类别】！")
            flag = int(input("是否全部统一修改（0是 1否）"))
            if flag == 1:
                modify_id = input("请指定待需要修改替换的类别id：")
                obj_label_id = input("请输入修改后的类别id：")
                print("将类别id:{} 替换为类别id:{}".format(modify_id, obj_label_id))
            elif flag == 0:
                modify_id = None
                obj_label_id = input("请输入修改后的类别id：")
            else:
                print('操作有误，请重新运行！')
                break
            modify_label(txt_list, obj_label_id, modify_id)
            
        elif mode == 'removeLabel':
            flag = int(input("执行【删除标签文件中某类别id】:0\n 执行【保留指定id】:1\n"))
            if flag == 0:
                remove_label_id = input("请指定需要排除的类别id：")
                remove_label_line(txt_list, remove_label_id)
            elif flag == 1:
                save_label_id = input("请指定需要保留的类别id：")
                remove_label_line(txt_list, save_label_id, True)

        elif mode == 'mapFile':
            print("执行【图片&标签文件匹配】")
            print('图片数量：{} 标签数量：{}'.format(len(img_list),len(txt_list)))
            print('start mapping')
            print('是否保存多余文件（0是 1否）')
            flag = int(input())
            if flag == 0:
                save_path = input('请输入文件保存路径：')
                map_file(img_list, txt_list, save_path)
            elif flag == 1:
                map_file(img_list, txt_list, None)

        elif mode == 'copyLableFile':
            
            print('执行【拷贝目标类别文件】')
            target_id_list = input('请输入要拷贝的目标类别id：')
            save_path = input('请输入文件保存的路径：')
            target_id_list = target_id_list.split(' ')
            copy_label_file(txt_list, target_id_list, save_path)
            
        elif mode == 'changeRoot':
            new_root = input("请输入新的操作源文件路径：")
            txt_list, img_list = get_files(new_root)
            print("路径修改成功：" + new_root)

        elif mode == 'exit':
            break

        else: 
            print("unkonw method!")
            
```



## 3.多文件夹内容拷贝

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

'''
@function: 拷贝不同文件夹下的文件到同一路径下
'''

import os
import argparse
from datetime import datetime
from tqdm import tqdm
import shutil
import multiprocessing as mp

parser = argparse.ArgumentParser(description='copy files')
parser.add_argument('--roots', type=str, help='操作源文件路径', default=r'')
parser.add_argument('--postfix', type=str, help='文件后缀', default=('.jpg', 'JPG', 'jpeg', '.JPEG', '.png', '.PNG', '.bmp', '.BMP', '.gif', '.GIF', '.txt'))
parser.add_argument('--save_path', type=str, help='保存路径', default=r'')
args = parser.parse_args()
print(args)

save_path = args.save_path

if not os.path.exists(save_path):
    os.mkdir(save_path)

# 获取指定后缀的文件列表
def get_files(roots, postfix):
    file_list = []
    for root, dirs, files in os.walk(roots):
        if len(files) == 0:
            continue
        for file in files:
            if file.endswith(postfix):
                file_list.append(os.path.join(root, file))
    return file_list

def copy_files(file):
    fileName = os.path.split(file)[-1]
    shutil.copy(file, os.path.join(save_path, fileName))

if __name__ == '__main__':
    start = datetime.now()
    files = get_files(args.roots, args.postfix)
    print('文件数量：{}'.format(len(files)))
    print('开始copy文件')
    pool = mp.Pool(processes=10)
    pool.map(copy_files, files)
    print('finish copy!')
    print('copy样本使用时间：[{}]'.format(datetime.now() - start))
```