# 时序逻辑
## 基本器件
<table>
    <tr>
        <td >
        <center>
        <img src="https://pic.imgdb.cn/item/669341d7d9c307b7e92e2fab.png" height = "100"/>
        <br/>SR锁存器电路图
        </center>
        </td>
        <td ><center><img src="https://pic.imgdb.cn/item/66934448d9c307b7e930b551.png" height = "100" />
        <br/>SR锁存器标志图
        </center></td>
    </tr>
</table>
<table>
    <tr> 
    <td> <center>
    <img src="https://pic.imgdb.cn/item/6693575ad9c307b7e946ac8d.png" alt="image.png">
    <br/>D锁存器
    </td> </center>
    </tr>
    <tr> 
    <td> <center>
    <img src="https://pic.imgdb.cn/item/66935898d9c307b7e948279d.png" alt="image.png">
    <br>D触发器
    </td> </center>
    </tr>
</table>

## 状态机
在对状态机进行编码时，二进制编码和独热码各有优劣。

二进制编码需要的寄存器更少，独热编码输出逻辑比较简单，所需的门电路可能更少

## 时序
![](https://pic.imgdb.cn/item/669894cbd9c307b7e9361a05.png)

- Contamination Time：表示一个信号受另一个信号影响开始发生变化的时间，即变化的最短时间
- Propagation Time：表示一个信号受另一个信号影响发生变化并保持稳定的时间，即变化的最长时间
- Setup Time：上升沿来之前被采样信号需要保持不变的最短时间
- Hold Time：上升沿之后被采样信号需要保持不变的最短时间

当被采样信号没有保持足够的$t_setup$或者$t_hold$时，输出是一种metastable，即亚稳态，亚稳态会经过一段时间$t_{res}$后变成稳定的0或1

消除亚稳态可以用如下结构的synchronizer，需要满足$T_c>t_{res}+t_setup$

![synchronizer](https://pic.imgdb.cn/item/66989e6cd9c307b7e93fcf5e.png)