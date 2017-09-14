# recommend
#Recommend related goods to the user
from numpy import *
import pandas as pd
import os
#生成个人——货物数据
def createGoods():
	os.chdir(r'C:\Users\Administrator.ZGC-20130623RFQ\Desktop')
	basket=pd.read_csv('BASKETS1n.csv')
	goods=basket.T[7:].T
	goods=goods.replace(['T','F'],[1,0])
	goods['cardid']=basket['cardid']
	return goods
#将数据转为字典
def df2dict(dataset):
	dictGoods={}
	for index,row in dataset.iterrows():
		if row['cardid'] not in dictGoods:
			dictGoods[row['cardid']]={i for i in row.index[:-1] if row[i]==1}
	return dictGoods
#goods=createGoods()
#print(df2dict(goods))
def reverseDict(users):
	goods_users=dict()
	for u,item in users.items():
		for i in item:
			if i not in goods_users:
				goods_users[i]=set()
			goods_users[i].add(u)
	return goods_users
#goods=createGoods()
#users=df2dict(goods)
#print(reverseDict(users))
def cAndNum(goods_users):
	C=dict()
	N=dict()
	for i,users in goods_users.items():
		for u in users:
			N.setdefault(u,0)
			N[u]+=1
			C.setdefault(u,{})
			for v in users:
				if u==v:
					continue
				C[u].setdefault(v,0)
				C[u][v]+=1
	return C,N
#goods=createGoods()
#users=df2dict(goods)
#goods_users=reverseDict(users)
#print(cAndNum(goods_users))
def userSimilarity(C,N):
	W=dict()
	for u,related_users in C.items():
		W.setdefault(u,{})
		for v,cuv in related_users.items():
			W[u].setdefault(v,0)
			W[u][v]+=cuv/sqrt(N[u]*N[v])
	return W
#goods=createGoods()
#users=df2dict(goods)
#goods_users=reverseDict(users)
#C,N=cAndNum(goods_users)
#print(userSimilarity(C,N))
def recommend(user,trains,W):
	rank=dict()
	items=trains[user]
	for u,wuv in W[user].items():
		for i in trains[u]:
			if i in items:
				continue
			rank.setdefault(i,0)
			rank[i]+=wuv
	return sorted(rank.items(),key=None,reverse=False)
def result():
	goods=createGoods()
	users=df2dict(goods)
	list_users=list(users.keys())
	goods_users=reverseDict(users)
	C,N=cAndNum(goods_users)
	for i in list_users:
		print("用户",i,"推荐：",recommend(i,users,userSimilarity(C,N)))
print(result())
