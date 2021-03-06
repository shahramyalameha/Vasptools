# Vasptools
some toolkits for VASP
## vaspeqstress.sh
小侯飞氚用python2写了VASP定压优化脚本，我之前用python3重写成vaspeqstress.py，但是我不喜欢python加模块的模式，而且服务器上常常不好自己配置python，当然也可能是我想多了。总之我用bash重新写了vaspeqstress.sh，不需要任何多余的模块，并且这样也可以很方便的用于作业管理系统。
## 晶格变形脚本的区别
deform.sh用于向POSCAR文件施加变形，seriesdeform.sh用于施加系列变形，idealdeform用于计算应力应变曲线。注意，以上三个脚本施加的变形都是基于晶体坐标系施加变形，也就是晶格的a b c分别对应[1 0 0] [0 1 0] [0 0 1]。另外三个带后缀2的脚本功能与前者完全相同，但是变形是在笛卡尔坐标系上，跟晶格坐标系没有关系。
## idealdeform.sh使用指南
+ 下面我开始讲一下脚本的使用方法，原理就不细讲了，根本在于假设应变过程足够慢，可以看作准静态应变，这样我们只需按一定应变间隔施加应变矩阵，优化晶胞和原子即可。首先我们需要准备优化晶胞所需要的VASP四个输入文件。注意，POSCAR的原子坐标必须是分数坐标 。INCAR中ISIF可以用4，但请在OPTCELL中关闭你变形方向的弛豫 (OPTCELL是什么？自己去DET群文件搜搜吧)。最好不要用3，晶格的体积发生变化会导致曲线失真。然后将idealdeform.sh放到同一文件夹。我测试的体系是fcc Ni, a为3.5058Å。
现在我们来修改脚本中相关参数。
![image](https://github.com/ponychen123/Vasptools/blob/master/image/1.png)
+ orientation表示施加应变的方式，你可以选择XX, YY, ZZ, XY, XZ, YZ六种变形方式中的一种，前三种为拉伸，后三种为剪切。initial为施加的初始应变，step表示每一步应变的间隔，num表示从初始应变开始按照step间隔一共连续施加了多少次应变。图中的例子就表示对fcc Ni沿着X轴拉伸，初始应变为0，每隔0.01施加应变进行一次拉伸，一共施加了100次应变，最后施加的应变为1.000(100%)。当然你也可以使用mystrain设置添加的应变矩阵，要施加的方向为非0，不施加的方向为0.如果mystrain被设置了，orientation会被忽略。mpiexec表示你的运行语句，由你的系统决定。最后直接运行脚本即可。
![image](https://github.com/ponychen123/Vasptools/blob/master/image/2.png)
+ 脚本会告诉你目前进行的进度，我们只需静待完成。计算完成后，程序默认会用gnuplot画一个工程应力应变曲线。
![image](https://github.com/ponychen123/Vasptools/blob/master/image/3.png)
+ 单位为GPa。我已经重新修改过单位，正号就表示拉应力。如果你觉得这样子看很辣眼睛，你可以将plot.sh也复制到当前文件夹，然后运行
![image](https://github.com/ponychen123/Vasptools/blob/master/image/4.png)
+ 脚本运行完后，工程应力应变曲线的数据会被保存到engineeringstressstrain.all中，真实应力应变曲线的数据会被保存到truestressstrain.all中。如果某一步中出现无法收敛的情况，程序会停止。你应该在OUTCAR中查找原因，并从当前阶段继续运算。
+ 如果要续算，只需要把initial值改成新的数值接着算就可以，程序会自动处理相关变化。

