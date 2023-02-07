---
title: "PCA PLOT"
description: 
date: 2023-01-31T16:58:12+08:00
hidden: false
comments: true
draft: true

---

想画个一维的图，最开始还在color那个地方尝试了query，报错说要用eval，最后发现直接写就行

{{< highlight python >}}
color_dict[lowD.query("PC1=="+str(lowD['PC1'][149]))["target"].values[0]]
{{< /highlight >}}

但是这个没有图例，而且color永远都是blue

{{< highlight python >}}
x = lowD['PC1']
y = np.zeros_like(x)

color_dict = {
    0: 'b',
    1: 'g',
    2: 'r',
}

plt.figure(figsize=(16, 3))
plt.plot(lowD['PC1'], y, ls='dotted',alpha = 0.4,color=color_dict[lowD[lowD['PC1']==x['target'].values[0]], lw=2)
plt.show()
{{< /highlight >}}


这个图color至少有三个了，但是图例是渐变颜色

{{< highlight python >}}
y = np.zeros_like(lowD['PC1'])
y=pd.DataFrame(y,columns=['y'])
data_plot = pd.concat([lowD, y], axis=1)


ax=data_plot.plot.scatter('PC1', 'y', c='target', colormap='jet',figsize = (16,3))
plt.show()
{{< /highlight >}}

这个终于ok了

{{< highlight python >}}
scatter_x = lowD['PC1'].values
scatter_y = np.zeros_like(lowD['PC1'])
group = lowD['target'].values
sdict = {0: 'setosa', 1:'versicolor', 2:'virginica'}
cdict = {0: 'red', 1: 'blue', 2: 'green'}
fig, ax = plt.subplots(figsize = (16,3))
for g in np.unique(group):
    ix = np.where(group == g)
    ax.scatter(scatter_x[ix], scatter_y[ix], c = cdict[g], label = sdict[g], s = 100)
ax.legend()
plt.show()
{{< /highlight >}}


二维的

{{< highlight python >}}
fig = plt.figure(figsize = (8,8))
ax = fig.add_subplot(1,1,1)
ax.set_xlabel('Principal Component 1', fontsize = 15)
ax.set_ylabel('Principal Component 2', fontsize = 15)
ax.set_title('2 Component PCA', fontsize = 20)
targets = [0, 1, 2]
species = ['setosa', 'versicolor', 'virginica']
colors = ['r', 'g', 'b']
for target, color in zip(targets,colors):
    indicesToKeep = lowD2['target'] == target
    ax.scatter(lowD2.loc[indicesToKeep, 'PC1']
               , lowD2.loc[indicesToKeep, 'PC2']
               , c = color
               , s = 50)
ax.legend(species)
ax.grid()
{{< /highlight >}}


其他问题：
pandas datafream 和 numpy array 相互转换:

{{< highlight python >}}
nparray = pddatafream.values
pddatafream = pd.DataFrame(pddatafream.nparray, columns=['col_name'])
{{< /highlight >}}

numpy array 拼接 (4,)to(4,2):
{{< highlight python >}}
two_egi=np.column_stack((eigVect1,eigVect2))
{{< /highlight >}}