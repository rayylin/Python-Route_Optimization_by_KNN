#Data preprocessing
---
	df = pd.read_csv("Total.csv")
	dic = {"state": [df.iloc[i][0] for i in range(df.shape[0])],
       		"stores": [df.iloc[i][1] for i in range(df.shape[0])],
       		"year": [2019 for i in range(df.shape[0])]}
       
	df_19 = pd.DataFrame(dic)

	dic = {"state": [df.iloc[i][0] for i in range(df.shape[0])],
	       "stores": [df.iloc[i][2] for i in range(df.shape[0])],
	       "year": [2020 for i in range(df.shape[0])]}
	df_20 = pd.DataFrame(dic)

	dic = {"state": [df.iloc[i][0] for i in range(df.shape[0])],
	       "stores": [df.iloc[i][3] for i in range(df.shape[0])],
	       "year": [2021 for i in range(df.shape[0])]}
	df_21 = pd.DataFrame(dic)


	#order is not important, so just append
	df_all = df_19.append(df_20.append(df_21))
	print(df_all)

	df_all.to_csv("totaldf.csv", index = True)



#Assign each state to the nearest DC
---
	import pandas as pd
	df = pd.read_csv("state_co_1.csv")
	#latitude and longitude
	dic_s = {df.iloc[i][0].split(",")[0]:[round(df.iloc[i][1],2),round(df.iloc[i][2],2)] for i in range(df.shape[0])} 
	#print(dic_s.items())
	wh = pd.read_csv("wh.csv") 
	#latitude and longitude of DCs
	dic_w = {wh.iloc[i][0]:[round(wh.iloc[i][1],2),round(wh.iloc[i][2],2)] for i in range(wh.shape[0])}
	print(dic_w.items())

#Calculate distance
---
	from haversine import haversine, Unit 
	
	distribute = {}
	for i,j in dic_s.items(): 
	    dis_dic = {}
	    hav = {} 
	    for k,l in dic_w.items():
		hav[k] = haversine(j,l) 	
	    #print(pd.DataFrame({i:hav.items()}))
	    distribute[i] = sorted(hav.items(), key = lambda x: x[1])[0]

	dic = {"state": [i for i in distribute.keys()],
	       "belong": [i[0] for i in distribute.values()],
	       "distance": [i[1] for i in distribute.values()]}

	table = (pd.DataFrame(dic))
	table.to_csv("belong_t.csv")


# Applied K-Means
---
	import numpy as np
	import matplotlib.pyplot as plt
	import random
	
	x = [round(df.iloc[i][1],2) for i in range(df.shape[0])] 
	y = [round(df.iloc[i][2],2) for i in range(df.shape[0])] 

	seed_num = 5  #num of DC
	dot_num = df.shape[0] 

	#Use first 5 as default DCs
	kx = x[:5] #[x[random.randint(0,len(x))] for i in range(seed_num)]#np.random.randint(0, 500, seed_num)
	ky = y[:5] #[y[random.randint(0,len(y))] for i in range(seed_num)]#np.random.randint(0, 500, seed_num)

	#print(f"{x}\n{y}\n{kx}\n{ky}")

	def dis(x, y, kx, ky):
	    return haversine([x,y],[kx,ky]) #如果不是經緯度的話這邊要換成((x0-x1)^2+(y0-y1)^2)^0.5

# clustering
---
	def cluster(x, y, kx, ky):

	    team = []

	    for i in range(seed_num):  
		team.append([])
	    mid_dis = 99999999  
	    for i in range(dot_num):
		for j in range(seed_num):
		    distant = dis(x[i], y[i], kx[j], ky[j])
		    if distant < mid_dis:
			mid_dis = distant
			flag = j
		team[flag].append([x[i], y[i]])
		mid_dis = 99999999
	    return team

	def re_seed(team, kx, ky):
	    sumx = 0
	    sumy = 0
	    new_seed = []
	    for index, nodes in enumerate(team):
		if nodes == []:
		    new_seed.append([kx[index], ky[index]])
		for node in nodes:
		    sumx += node[0]
		    sumy += node[1]
		new_seed.append([int(sumx / len(nodes)), int(sumy / len(nodes))])
		sumx = 0
		sumy = 0
	    nkx = []
	    nky = []
	    for i in new_seed:
		nkx.append(i[0])
		nky.append(i[1])
	    return nkx, nky

# k-means 
---
	def kmeans(x, y, kx, ky, fig):
	    team = cluster(x, y, kx, ky)
	    #print(team)
	    nkx, nky = re_seed(team, kx, ky)

	    # plot: nodes connect to seeds
	    cx = []
	    cy = []
	    line = plt.gca()
	    for index, nodes in enumerate(team):
		# print(index,nodes)
		for node in nodes:
		    cx.append([node[0], nkx[index]])
		    cy.append([node[1], nky[index]])
		for i in range(len(cx)):
		    line.plot(cx[i], cy[i], color='r', alpha=0.6)
		cx = []
		cy = []
	    # Visualize
	    feature = plt.scatter(x, y)
	    k_feature = plt.scatter(kx, ky)
	    nk_feaure = plt.scatter(np.array(nkx), np.array(nky), s=50)
	    plt.show()

	    # stop criteria
	    if nkx == list(kx) and nky == (ky):
		return team 
	    else:
		fig += 1
		return kmeans(x, y, nkx, nky, fig)

	team = (kmeans(x, y, kx, ky, fig=0))
	#print(team)

#Final Result
---
	ind_l = []
	city_l = []

	for ind, item in enumerate(team): 
	    for j in item:        
		# print(j[0])
		for i in range(len(x)): 
		    if j[0] == x[i] and j[1]==y[i]: 
			ind_l.append(ind)
			city_l.append(df.iloc[i][0].split(",")[0])

	dic = {"state": [city_l[i] for i in range(len(city_l))],
	       "index": [ind_l[i] for i in range(len(city_l))],
	       "coord": [[x[i],y[i]] for i in range(len(city_l))]}
	clus = (pd.DataFrame(dic))
	clus.to_csv("clus.csv")


