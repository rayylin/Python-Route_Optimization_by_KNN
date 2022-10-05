# GAP 0608

```
資料前處理, 把年份拆開
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
#因為順序不會影響索性直接三份疊加
df_all = df_19.append(df_20.append(df_21))
print(df_all)
df_all.to_csv("totaldf.csv", index = True)
```

![Untitled](GAP%200608%204b5071e3b7dc46d19cb42d4d94e8b100/Untitled.png)

![Untitled](GAP%200608%204b5071e3b7dc46d19cb42d4d94e8b100/Untitled%201.png)

工廠分布狀況, 家數(如果能有實際產能會更好

![Untitled](GAP%200608%204b5071e3b7dc46d19cb42d4d94e8b100/Untitled%202.png)

店面分布狀況(州), 如果有銷量會更好

通常要去規劃這種東西要知道銷量, 運輸成本, 持有成本, 倉儲容量跟進貨點, 前置時間(Leadtime)等等

```
#這邊要去算每州跟哪個倉儲中心最近, 指派給那個中心負責
import pandas as pd
df = pd.read_csv("state_co_1.csv")
#原始資料是 IN, US 所以用","分割並取第一各值會得到IN
#會得到"IN":[x,y] 某個州的經緯度
dic_s = {df.iloc[i][0].split(",")[0]:[round(df.iloc[i][1],2),round(df.iloc[i][2],2)] for i in range(df.shape[0])} #經緯度太長, 取到小數點兩位就好
#print(dic_s.items())

wh = pd.read_csv("wh.csv") #物流中心座標, 同樣是key是地名value經緯度
# print(wh.iloc[1][1])
dic_w = {wh.iloc[i][0]:[round(wh.iloc[i][1],2),round(wh.iloc[i][2],2)] for i in range(wh.shape[0])}
print(dic_w.items())

from haversine import haversine, Unit #用經緯度來算座標的函式庫
distribute = {}
for i,j in dic_s.items(): #每個key, value pair
    dis_dic = {}
    hav = {} #空字典, 在每州都要清空
    for k,l in dic_w.items():
        hav[k] = haversine(j,l) #丟進兩個座標會回傳距離
				#內層的迴圈是在做 把每州到每個倉儲的距離算出來
    #print(pd.DataFrame({i:hav.items()}))
    distribute[i] = sorted(hav.items(), key = lambda x: x[1])[0]
#這邊在做的是去排序然後把每州對到最近的倉儲給貼近字典(哪個倉儲, 距離多遠)

dic = {"state": [i for i in distribute.keys()],
       "belong": [i[0] for i in distribute.values()],
       "distance": [i[1] for i in distribute.values()]}

table = (pd.DataFrame(dic))
table.to_csv("belong_t.csv")
```

第一張是五個倉儲(沒有德州)

![Untitled](GAP%200608%204b5071e3b7dc46d19cb42d4d94e8b100/Untitled%203.png)

下面這張包含longview, TX

![Untitled](GAP%200608%204b5071e3b7dc46d19cb42d4d94e8b100/Untitled%204.png)

這邊可以思考一下, 五個倉儲點的risk pooling 效果一定比較好; 為什麼要多加德州一個點?

實作KNN, 去算中心點並做迭代

```
#看不懂的計算可以不要理, 反正未來也不一定會用到
import numpy as np
import matplotlib.pyplot as plt
import random
# 群集中心和元素的數量
x = [round(df.iloc[i][1],2) for i in range(df.shape[0])] #所有經度
y = [round(df.iloc[i][2],2) for i in range(df.shape[0])] #所有緯度, 或相反XD

seed_num = 5  #要分幾群, 最好不要在下面用數字, 不然要改就要一個一個找
dot_num = df.shape[0] #shape[0]取長度

# 初始群集中心
#本來想要隨機配對, 但後來想想好像配出來一個不存在的組合好像也沒什麼意義, 於是取前五個經緯度
kx = x[:5] #[x[random.randint(0,len(x))] for i in range(seed_num)]#np.random.randint(0, 500, seed_num)
ky = y[:5] #[y[random.randint(0,len(y))] for i in range(seed_num)]#np.random.randint(0, 500, seed_num)
# print(f"{x}\n{y}\n{kx}\n{ky}")

# 兩點之間的距離
def dis(x, y, kx, ky):
    return haversine([x,y],[kx,ky]) #如果不是經緯度的話這邊要換成((x0-x1)^2+(y0-y1)^2)^0.5

# 對每筆元素進行分群
def cluster(x, y, kx, ky):
    team = []
    for i in range(seed_num):  #這邊原本是3, 寫程式的時候這種操控變數最好不要直接打; 不然要改會麻煩的要死
        team.append([])
    mid_dis = 99999999  #這邊用一個很大的數字來做初始值, 如果距離都超大那這邊就會錯
    for i in range(dot_num):
        for j in range(seed_num):
            distant = dis(x[i], y[i], kx[j], ky[j])
            if distant < mid_dis:
                mid_dis = distant
                flag = j
        team[flag].append([x[i], y[i]])
        mid_dis = 99999999
    return team

# 對分群完的元素找出新的群集中心
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

# k-means 分群
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
    # 繪圖
    feature = plt.scatter(x, y)
    k_feature = plt.scatter(kx, ky)
    nk_feaure = plt.scatter(np.array(nkx), np.array(nky), s=50)
    plt.show()

    # 判斷群集中心是否不再更動
    if nkx == list(kx) and nky == (ky):
        return team #這邊原本是只有return
    else:
        fig += 1
        return kmeans(x, y, nkx, nky, fig)
				#這邊原本沒有return, 改成遞迴的格是因為我最後要回傳值
				#如果不回傳下面那行team無法取得(可以把team 設在外面但這不是一個好方法
				#因為能改的地方越多程式出錯的機會就越高也越難抓
team = (kmeans(x, y, kx, ky, fig=0))
#print(team)
#這邊的格式是, 某個群及所包含的座標點 i:[(x1,y1),(x2,y2)....] 

#下面要再把座標點轉換回城市, 屬於哪個群
ind_l = []
city_l = []
#可以用dataclass等方式, 三層迴圈的效能會很糟糕(但五十州而已有點懶
for ind, item in enumerate(team): #第ind群, item代表所有座標點
    for j in item:        #j是某個城市的經緯度
        # print(j[0])
        for i in range(len(x)): #對所有州而言
            if j[0] == x[i] and j[1]==y[i]: #如果和某州xy座標都相等, 代表有對應到
                ind_l.append(ind)
                city_l.append(df.iloc[i][0].split(",")[0])

dic = {"state": [city_l[i] for i in range(len(city_l))],
       "index": [ind_l[i] for i in range(len(city_l))],
       "coord": [[x[i],y[i]] for i in range(len(city_l))]}
clus = (pd.DataFrame(dic))
clus.to_csv("clus.csv")
```

過程大概就在做這個, 去計算怎麼分群最好

算距離一樣用havershine, 因為那些值是經緯度不能直接相減平方在開根號…吧?

![Untitled](GAP%200608%204b5071e3b7dc46d19cb42d4d94e8b100/Untitled%205.png)

從下面這張圖應該不難發現longview其實真的是很適合呢~

![Untitled](GAP%200608%204b5071e3b7dc46d19cb42d4d94e8b100/Untitled%206.png)

隨機分群可能會有個問題, 某個中心點可能根本沒有倉儲中心

甚至上面這張圖的第四群(粉紅色)的中心WY好像根本沒有店

follow up 

用Gene, Simulated annealing等方法嘗試找出risk pooling 最佳解