```
# coding: utf-8

# In[1]:

import osmapi
import xml.etree.ElementTree as ET
import numpy as np
import pandas as pd
import pprint
from collections import defaultdict
from collections import Counter
import re
import codecs
import csv
import sqlite3

# In[ ]:
# Map Area
# Seoul Metro area
# https://s3.amazonaws.com/metro-extracts.mapzen.com/seoul_south-korea.osm.bz2

# In[8]:
# raw osm data file

OSMfile ='/Users/JAY/Desktop/Udacity/Project3/seoul_south-korea.osm'

# In[9]:
# Read OSMfile

tree=ET.parse(OSMfile)
root=tree.getroot()

# In[10]:
# Number of tags for node, relation, bounds and way in Seoul map
# Set vacant dict.

tags ={}

tags['bounds'] =0
tags['node']=0
tags['way']=0
tags['relation']=0

# In[11]:
# Number of tags for node, relation, bounds and way

for a in root:
    tags[a.tag] = 0

# In[12]:
# Allocate the Number of tags for node, relation, bounds and way

for a in root:
    if 'bounds' in a.tag:
        tags['bounds']+=1
    elif 'node' in a.tag:
        tags['node'] +=1
    elif 'way' in a.tag:
        tags['way'] +=1
    elif 'relation' in a.tag:
        tags['relation'] +=1
    else:
        print a.tag

print tags

# In[13]:
# Attrib 'k' value in 'tag' tag

tag_k =[]
tag_v =[]

# Loop attrib 'k' value in each list 'tag_k' and 'tag_v'

for a in root:
    for tag in a.findall('tag'):
        tag_k.append(tag.attrib['k'])
        tag_v.append(tag.attrib['v'])

# In[14]:
# DATA Wrangling, Cleaning

#     "# upper characters change to lower char\n",
#     "# korean k value\n"
#     "# address 'v' value typed  v : 'Korean' ( 'English' ) >>> just  v : 'English'
#     "# 'v' value street name change. gil >> -gil, ro >> -ro

# In[15]:
# list all set of 'k' value in 'tag' tag

uniq_tag_k = list(set(tag_k))
# print uniq_tag_k[0:5]

# In[16]:
# count 'k' values how often they are used

tag_k_numbers = {}

# for a in tag_k:
#     tag_k_numbers[a] = 0

for a in tag_k:
    if a in uniq_tag_k:
        tag_k_numbers[a] += 1
    else:
        tag_k_numbers[a] = 1

# In[17]:

# ************** Additional Problems encountered during the data process **************
# ncat:bridge >> bridge
# bridge:name:ko >> bridge
# bridge:name:ko_rm >> bridge
# building:levels >> building
# name:ko >> name:ko
# source:bridge >> bridge
# bridge:name:en >> bridge
# route_ref >> route
# name:en >> name
# addr:postcode >> postcode
# bridge:name >> bridge
# name:kr_rm >> name:ko


# print tag_k_numbers
# tag k value mentioned 1000 times more than, then print them out with the frequency of the k value

for i in tag_k_numbers:
    if tag_k_numbers[i]>1000:
        print '%s : %i' %(i, tag_k_numbers[i])

# In[18]:

# iterparse equals line by line parse
# count tag frequency

def count_tags(filename):
    counts = defaultdict(int)

    for event, elem in ET.iterparse(filename):
        counts[elem.tag] += 1

    return counts

# In[19]:
ct = count_tags(OSMfile)

# In[20]:
# count number of k value in 'tag' tag

def count_tags_k(filename):
    counts = defaultdict(int)

    for event, elem in ET.iterparse(filename):
        if elem.tag == 'tag':
            counts[elem.attrib['k']] += 1

    return counts

# In[21]:
# count number of k value in 'tag' tag
ctk = count_tags_k(OSMfile)

# In[22]:
# Data filtering compiler

#lower case
lower = re.compile(r'^([a-z]|_)*$')

#colon
lower_colon = re.compile(r'^([a-z]|_)*:([a-z]|_)*$')

#problem character
problemchars = re.compile(r'[=\+/&<>;\'"\?%#$@\,\. \t\r\n]')

#ignore the case
street_type_re = re.compile(r'\b\S+\.?$', re.IGNORECASE)

# In[23]:

# count number of string in accordance with the search compiler above
# count number of lower case, lower colon, problemchars, and others

def count_k_condition(filename):
    # counts = defaultdict(int)
    counts = {"lower" : 0, "lower_colon" : 0, "problemchars" : 0, "other" : 0}

    for event, elem in ET.iterparse(filename):
        if elem.tag == 'tag':
            low_re = lower.search(elem.attrib['k'])
            lower_colon_re = lower_colon.search(elem.attrib['k'])
            problemchars_re = problemchars.search(elem.attrib['k'])

            if low_re:
                counts["lower"] +=1
            elif lower_colon_re:
                counts["lower_colon"] +=1
                # print lower_colon_re
            elif problemchars_re:
                counts["problemchars"] +=1
                # print elem.attrib['k']
            else:
                counts["other"] +=1

            # counts[elem.attrib['k']] += 1

    return counts

# In[24]:
ck_condition = count_k_condition(OSMfile)


# In[25]:
# Make CSV file from OSM which revised with 'v' value in 'Node' and 'Way' tag

NODE_FIELDS = ['id', 'lat', 'lon', 'user', 'uid', 'version', 'changeset', 'timestamp']
# NODE_TAGS_FIELDS = ['id', 'key', 'value', 'type']

WAY_FIELDS = ['id', 'user', 'uid', 'version', 'changeset', 'timestamp']
# WAY_TAGS_FIELDS = ['id', 'key', 'value', 'type']

node_elem = {}
node_tag = {}
node_tag_list_v = []
node_group = {}
node_tag_group = {}
node_group_list_v = []
way_group_list_v = []
way_node_group_list_v =[]


# In[26]:
# Revised the problems encountered during the data wrangling
# Firstly,
# 'v' value street name change. gil >> -gil, ro >> -ro
# for example, normally used Wall-street, 34th-street in KOREA (but It was typed Wallstreet, 34thstreet)
# value change

mapping = {"gil": "-gil", "ro": "-ro"}
expected = ["-gil", "-ro"]

a = street_type_re.search(expected[0])
# a = street_type_re.search(expected[1])

print a.group()

# j = 0

def replace_value(name, mapp):

    if re.search('-gil', name):
        # print i
        pass
    else :
        if re.search('gil', name):
            # print i
            name  = re.sub('gil', '-gil', name)
            # print i
    # gil_list.append(i)
    koen = KorToEng(name)

    return koen


# In[27]:

word = []
words = []

# Revised the problems encountered during the data wrangling
# Secondly,
# address 'v' value typed  v : 'Korean' ( 'English' ) >>> just  v : 'English'
# value change

def KorToEng(data):

    if len(re.split(r'[()]',data)) > 1:
        word = len(re.split(r'[()]',data))
        words = re.split(r'[()]', data)
        if word == 3:
            data = words[1]
    else:
        pass

    return data


# In[28]:

# Make 'node' tag dictionary to make CSV file

def make_dic_node(dataname):

    tree = ET.parse(dataname)
    root = tree.getroot()

    # for event, elem in ET.iterparse(dataname):
    for elem in root:
        node_elem = {}
        node_group = {}
        node_tag = {}
        node_tag_list_v = []

        if elem.tag == 'node':
            for idx in NODE_FIELDS:
                node_elem[idx] = elem.attrib[idx]

            node_group['node'] = node_elem

            # node_group_llist.append(node_group)
            for idy in elem.iter('tag'):
                node_tag = {}
                node_tag['id'] = node_elem['id']
                node_tag['key'] = idy.attrib['k']
                node_tag['value'] = replace_value(idy.attrib['v'],mapping)

                node_tag_list_v.append(node_tag)

            node_group['node_tag'] = node_tag_list_v

            node_group_list_v.append(node_group)

    return node_group_list_v


# In[29]:

# Make 'way' tag dictionary to make CSV

def make_dic_way(dataname):

    tree = ET.parse(dataname)
    root = tree.getroot()

    for elem in root:
        way_elem = {}
        way_group = {}
        way_tag = {}
        way_tag_list_v = []
        way_node_group_list_v = []

        if elem.tag == 'way':
            j = 0

            for idx in WAY_FIELDS:
                way_elem[idx] = elem.attrib[idx]

            way_group['way'] = way_elem

            # for idz in elem.iter('nd'):


            for elem_v in elem:

                if elem_v.tag == 'nd':
                    way_node = {}
                    way_node['id'] = way_elem['id']
                    way_node['node_id'] = elem_v.attrib['ref']
                    way_node['position'] = j

                    way_node_group_list_v.append(way_node)

                    j = j + 1

                way_group['way_node'] = way_node_group_list_v

                if elem_v.tag == 'tag':
                # for idy in elem.iter('tag'):
                # # for idy in elem.iter('tag'):
                    way_tag = {}
                    way_tag['id'] = way_elem['id']
                    way_tag['key'] = elem_v.attrib['k']
                    way_tag['value'] = replace_value(elem_v.attrib['v'],mapping)

                    way_tag_list_v.append(way_tag)

                # way_group['way_node'] = way_node_group_list_v
                way_group['way_tag'] = way_tag_list_v

            way_group_list_v.append(way_group)

    return way_group_list_v


# In[30]:

nd_group_node = make_dic_node(OSMfile)


# In[31]:

nd_group_way = make_dic_way(OSMfile)


# In[32]:

print(nd_group_node[40000:40001])
# print(nd_group_node[40100:40150])
# print(nd_group_way[40000])


# In[33]:

print(nd_group_way[40000:40001])


# In[34]:

# Dictionary data converted to CSV

with open('node_group.csv','w') as f:
    w = csv.writer(f)

    node_key = nd_group_node[0]['node'].keys()
    w.writerow(node_key)

    for i in np.arange(len(nd_group_node)):
        a_node = nd_group_node[i]['node'].values()
        a_node = [en.encode('utf-8') for en in a_node]
        w.writerow(a_node)


# In[35]:

with open('node_tag_group.csv','w') as h:
    w_ntg = csv.writer(h)

    node_tag_key = ['value','id','key']
    w_ntg.writerow(node_tag_key)

    for j in np.arange(len(nd_group_node)):
        # if isinstance(nd_group_node[j]['node_tag'], unicode):
        if not nd_group_node[j]['node_tag']:
            pass
        else:
            for k in nd_group_node[j]['node_tag']:
                a_tag = k.values()
                a_tag = [en_k.encode('utf-8') for en_k in a_tag]
                w_ntg.writerow(a_tag)


# In[36]:

with open('way_group.csv','w') as p:
    w_wg = csv.writer(p)

    way_key = nd_group_way[0]['way'].keys()
    w_wg.writerow(way_key)

    for i in np.arange(len(nd_group_way)):
        a_way = nd_group_way[i]['way'].values()
        a_way = [en.encode('utf-8') for en in a_way]
        w_wg.writerow(a_way)


# In[37]:

with open('way_tag_group.csv','w') as o:
    w_wtg = csv.writer(o)

    way_tag_key = ['value','id','key']
    w_wtg.writerow(way_tag_key)

    for j in np.arange(len(nd_group_way)):
        # if isinstance(nd_group_node[j]['node_tag'], unicode):
        if not nd_group_way[j]['way_tag']:
            pass
        else:
            for k in nd_group_way[j]['way_tag']:
                a_tag = k.values()
                a_tag = [en_k.encode('utf-8') for en_k in a_tag]
                w_wtg.writerow(a_tag)


#######################################################################
# Four csv files created so far.
# node_group.csv
# node_tag_group.csv
# way_group.csv
# way_tag_group_csv


# File Size

# seoul_south-korea.osm ........284.1MB
# new.db .......................319MB
# node_group.csv ...............107.5MB
# node_tag_group.csv............11.3MB
# way_group.csv ................8.7MB
# way_tag_group.csv ............9.3MB


# In[38]:

# sqlite3.version


# In[39]:

# sqlite3.sqlite_version

# In[41]:

###########################################################

# From now on, Run sqlite3

## Run sqlite3 on terminal
## look for the location of the project address
## make new database

# sqlite3 new.db

## import csv data created from python

# .mode csv
# .import way_tag_group.csv wayTagGroup
# .import way_group.csv wayGroup
# .import node_tag_group.csv nodeTagGroup
# .import node_group.csv nodeGroup

## check the table

##  input
# .table

## output
# nodeGroup     nodeTagGroup  wayGroup      wayTagGroup


# In[42]:

## node 'tag' tag value the most frequently presented on the map

## Input

# select value, count(*) as num from nodeTagGroup
# group by value
# order by num desc
# limit 10 ;

## Output

# http://kr.open.gugi.yahoo.com|17377
# mp2osm|11622
# 교차로및지명|11618
# bus_stop|9476
# yes|6850Va
# hospital|5093
# 병원|5001
# tower|3858
# Bing|2893
# subway_entrance|2510


## merge two tag value table. nodeTagGroup and wayTagGroup. to see which word the most frequently presented on the map
## create new table and insert two tables to newTable

## Input

# create table newTable (value, id, key);
# Insert into newTable (value, id, key)
# select (value, id, key) from nodeTagGroup;
# Insert into newTable (value, id, key)
# select (value, id, key) from wayTagGroup;

## Total Tag counts
# sqlite> select count(*) from newTable
#    ...> ;
# 640767

# select value, count(*) as num from newTable
# group by value
# order by num desc
# limit 20;

## Output

# yes|43170
# residential|40967
# http://kr.open.gugi.yahoo.com|17428
# service|15947
# no|12025
# mp2osm|11878
# 교차로및지명|11876
# bus_stop|9476
# commercial|8952
# 1|7565
# NTIC|7498
# secondary|6041
# apartments|5654
# hospital|5273
# footway|5116
# 병원|5005
# unclassified|4942
# Bing|4563
# tertiary|4130
# school|3914


# ************ Further research parts ***********
# ************ I don't know why there are so many 'yes' values on the map.
# ************ residential, service could be top 2 and 4 because so many people are using them.
# ************ don't know http://kr.open.gugi.yahoo.com even I couldn't open it. just poped up yahoo domain.

### What are the unique users ID

## INPUT

# sqlite> select user, count(*) as num from userTable
#    ...> group by user
#    ...> having num = 1;

## Total Node counts

# sqlite> select count(*) from userTable
#    ...> ;
# 1443573

## OUTPUT

# 2hertz|1
# 8iseoii|1
# Agmenor|1
# Amigos_Conmigo|1
# Ashs1126|1
# BBIBBIBi|1
# BG2YF|1
# BarrettHC|1
# Boppet|1
# BugBuster|1
# CG Kang|1
# CeteraTolle|1
# Christian Westhoff|1
# Chunsoo Lee|1
# CloCkWeRX|1
# DHKIM|1
# Dahee Choe|1
# Deen|1
# Dio Seo|1
# Dongyi|1
# Edatmpa|1
# Enzin333|1
# Ethan Lee|1
# Fallenstar|1
# Fumiko|1
# Gamayun|1
# GarionPark|1
# GercoKees|1
# Gilbert C|1
# Gyeongmoon Eom|1
# HHCacher|1
# Hankie|1
# Hero Kim|1
# Hojin Shin|1
# HoloDuke|1
# HorribleChoice|1
# Hue Lee|1
# HwangJinWoo|1
# Ikjune Yoon|1
# JISU|1
# JK_KOREA|1
# JS Kim|1
# Jae Sung Han|1
# Jang_ys|1
# Jean Kim|1
# Jedrzej Pelka|1
# Jewon|1
# Jochen Topf|1
# June Rainberry|1
# Junios|1
# Junmo Kim|1
# K83|1
# Kang Seung sik|1
# Kimbabchapati|1
# Lee Sehoon|1
# Lee jae hyung|1
# Louis JANG|1
# Luis36995_labuildings|1
# M31sa|1
# MAPconcierge|1
# Math1985|1
# Matthewmayer|1
# Maverick LEE|1
# MinSeok|1
# NTUKLC|1
# Nam Wook Kim|1
# None 1|1
# Nuk34|1
# Oberpfälzer|1
# Orca0418|1
# Pelcos|1
# Philogrammer|1
# Phrast|1
# RailMapper|1
# Rainbow International School|1
# RehobothGlobal|1
# Rodrigo Rega|1
# Rouni|1
# SGPark|1
# SNJeong|1
# S_young|1
# Sandi Khim|1
# Sangsun Park|1
# Sean Lee|1
# Shane Metzger|1
# SomeRandomMe|1
# StellarKoala|1
# Sunny Park|1
# TJ Lee|1
# TJJU|1
# Ted Kwon|1
# Tribunal Supremo Electoral|1
# U20|1
# VisualDive|1
# VlIvYur|1
# Wikidata|1
# Will3301|1
# YCH|1
# Yong-Chan Kim|1
# ZeboStoneleigh|1
# ajashton|1
# angys2|1
# arthur8|1
# asdf2|1
# awesomekimbos|1
# bes_internal|1
# bioshome|1
# chorani81|1
# chris johnston|1
# coolx|1
# cricket|1
# crucialsoft2012|1
# dalee02|1
# davidjudekim|1
# dbdbdpdp72|1
# dcs|1
# devmario|1
# disBtopia|1
# dk_bm|1
# dp2743gw|1
# dua804|1
# elucian|1
# emersion|1
# engiYong|1
# etajin|1
# falcon31|1
# gabyul|1
# geruto|1
# gsa|1
# gtggtg|1
# hanlong|1
# hansumo81|1
# hater|1
# heavens|1
# hgfonseca|1
# hkucharek|1
# hyeri jung|1
# hynho|1
# iamphoto|1
# idasinc|1
# ideacactus|1
# ireneNK|1
# itisjune|1
# iymlmsh|1
# jc67|1
# jdinjdi|1
# jee1033|1
# jengelh|1
# jeonseik|1
# jhparker|1
# jinalfoflia|1
# jjspride|1
# jong b|1
# jongho84|1
# jongseon son|1
# joyamor|1
# kangdaejang|1
# kjh1592|1
# kortinu|1
# labsurde|1
# lee kn|1
# leeSS|1
# leejeongjun|1
# legotown|1
# mailiam|1
# mapfreak|1
# mbncg|1
# mhu283|1
# mihaii_telenav|1
# miurahr|1
# moningbel|1
# monsterlee48|1
# moren|1
# moyogo|1
# mozolondon|1
# mtmail|1
# mubyoung|1
# myforest|1
# myjoey|1
# pierre1011|1
# popcornhouse|1
# ptateosi|1
# qordytpq|1
# raiseone|1
# raymondnewby|1
# redsnow78|1
# robleiphart|1
# rsnaru|1
# samiupsala|1
# sangukim|1
# sarojaba|1
# schumyHH|1
# seiwonKim|1
# selena0803|1
# seogain|1
# sh221b|1
# shiftkey|1
# sk92129|1
# smitchell78|1
# taijin|1
# tcochr|1
# techsavvy|1
# temip|1
# tillluna|1
# tlacoyo|1
# toshilow|1
# triumph47|1
# ukaszyk|1
# us38914|1
# user_5359|1
# vectorking|1
# voland|1
# wallasoft|1
# widelake|1
# wieland|1
# won21kr|1
# woo seungwan|1
# wuchoi|1
# yesbento|1
# yiqingj|1
# ypid|1
# yurasi|1
# zcom|1
# zorque|1
# Дмитрий Александров|1
# عقبة بن نافع|1
# 陳信翰|1
# 가면술사|1
# 거친마루|1
# 구정모|1
# 권중규|1
# 귀신시바|1
# 김석용|1
# 김설기|1
# 김송권|1
# 김형태|1
# 류재관|1
# 박종태|1
# 산티니 전국대리점 현황|1
# 서울대GIS실습계정002|1
# 서울대공간정보교육001|1
# 설레임|1
# 성안교회|1
# 양갱좋앙|1
# 어디서본아저씨|1
# 얼린사과|1
# 예림예|1
# 온천거북|1
# 우린굉장해|1
# 윤오리|1
# 의정부침례교회|1
# 이의재|1
# 정성태|1
# 최종철|1
# 카리브레스토랑|1
# 토토로검도|1
# 푸른하늘|1
# 하늘여행|1
# 한양내과|1


# what are the user ID the most contributed on the KOREAN map
# Same as above, tried to make new table for nodeGroup and wayGroup
# Find the user name top 20

## Input

# create table userTable(chnageset, uid, timestamp, lon, version, user, lat, id);

# insert into userTable(chnageset, uid, timestamp, lon, version, user, lat, id)
# select changeset, uid, timestamp, lon, version, user, lat, id from nodeGroup;
# insert into userTable(chnageset, uid, timestamp, version, user, id)
# select changeset, uid, timestamp, version, user, id from wayGroup;

# select user, count(*) as num from userTable group by user order by num desc limit 20;

## Output

# maphunter36|149684
# 전기곰돌|144133
# cyana|80553
# IRTC1015|80090
# panhoong|54795
# icemango|38759
# CitymapperHQ|37556
# Lantian|33722
# Exj　|33338
# sanha|30836
# GPIOIPG|28909
# dabeeo|27901
# alimamo|24959
# asdfqwer51|23930
# ulil|21158
# namuori|20017
# thbz|17579
# MapPlus|17321
# Keonsoon Hwang|17271
# Albertus Liberius|16426


# In[43]:

# Conclusion

# As I look through the Seoul city data, there are no consistency and make me confusing sometimes. For example, the newTable which have key and value
# have to be divided by category. value meant to be specific name of something and key meant to be categorical name of value something like if value is
# Timesqure, then key should be tourist attraction or place something like this way.
#
# This I just took value, key from some last rows
#
# residential|landuse
# 예정|name
# residential|highway
# residential|highway
# 숭인동1나길|name
#
# It is a little bit unclarified difference between key and value, residential, high way. some korean characters and name (should be changed to upper category such like street name or something else)
# So, I am thinking it will be going to be more easier for users to recognize what each values mean when the map contributors use categorical values thoroughly.
# not just like, name or address. change them to what name or what address such like street name or school address.
# It could be great for using local data scientist to correct the wrong assigned name or values on SEOUL OSM data and give them a coupon like starbucks
# or amusement park when they devoted something on the SEOUL map. Even the franchise restaurant owner could let data scientist to work on their
# Restaurant information on the OSM data and give them some advantage when they use the restaurant. It could be good for both owner and the scientist just
# for a quick commitment
#
# In userTable, Tag value presented too many confusing language selection such as tag value containing korean, english, japanese, chinese and something else.
# specially, I think it would be better just using one or two most recognized languages in Asia, Korean and english
# It would be better to change the form in order to make it easy to recognize for both korean and english.
# There are still things needed to change (make things easy to be recognized)
#
# bridge:name:en >> bridge
# route_ref >> route
# name:en >> name
# addr:postcode >> postcode
# bridge:name >> bridge
# name:kr_rm >> name:ko
#
# Even I changed the name ro, gil to -ro, -gil but
# there are still exceptional cases even though they are a few. Also top one and two of the tourism which were hotel and motel were pretty surprising.
# I think people needed to update more tourism on the map. there are many places to visit as I said cooperated with local computer science students who
# can develop the detailed information and give them a reward.


# In[43]:




# In[ ]:

```
