---
title: 灰色模型的简单Java实现
date: 2017-08-19 16:50:29
tags: 
	- 笔记
---

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>


前几天在以前的遗留代码中发现一个问题，就是我生成的一个数据的走势曲线的预测值(用于灰色时间序列预测)总是和老代码里的不一致，具体来说就是：遗留代码里面的预测值的斜率总是为零，相比之下我生成的就比较合理了，和原有数据的走势相比基本一致。后来发现这个不知道谁写的代码里面又全是坑，连灰色模型都是错的，输入的数据也是错的。。。

所以，趁着这个周末有空，来写一些关于灰色模型的Java的简单实现。

### 一、对灰色模型的建模分析
首先，关于灰色模型（后面简称为GM），简单讲就是介于黑色模型和白色模型之间，利用已知的数据来推测未知数据的一种模型（更具体的描述可以看[灰色模型的简单介绍](http://wiki.mbalib.com/wiki/%E7%81%B0%E8%89%B2%E9%A2%84%E6%B5%8B%E6%B3%95)）。

好的，从上面我们可以知道，运用GM，有这么几个步骤：
1. 处理数据，使其可以运用于GM中

    设原始数据列为$$X^{(0)}=x^{(0)}(1)+x^{(0)}(2)+x^{(0)}(3)+...+x^{(0)}(n)$$那么其级数比则是：$$λ(k)=\frac{x^{(0)}(k-1)}{x^{(0)}(k)},k=2,3,4...n$$
    当所有级数比都在区间:$$(e^{\frac{-2}{n+1}},e^{\frac{2}{n+1}})(注:这个区间和正态分布的概率密度有关)$$
    此时的数据列就可以建立GM，否则，对原始数据列进行变换（比如平移），使其满足上面的条件。

2. 建立GM模型

    假设数据列:$$X^{(0)}=x^{(0)}(1)+x^{(0)}(2)+.。。+x^{(0)}(n)$$满足条件，我们再通过累加生成数据列(也可以使用其他方式，比如累减、均值等):$$X^{(1}=x^{(1)}(1)+x^{(1)}(2)+.。。+x^{(1)}(n)$$
    其中:$$x^{(1)}(k)=\sum_{i=0}^{k}x(i)$$
    定义\\(x^{(1)}\\)的灰导数为：
    $$dx^{(1)}(k)=x^{(0)}(k)=x^{(1)}(k)+x^{(1)}(k-1)$$
    令:$$z^{(1)}(k)=\frac{1}{2}(x^{(1)}(k)+x^{(1)}(k-1)),(k=2,3,...,n)$$
    定义GM(1,1)的灰微分方程模型为:
    $$x^{(0)}(k)+az^{(1)}(k)=b$$
    \\(其中x^{(0)}(k)称为灰导数，a称为发展系数，z^{(1)}(k)称为白化背景值，b称为灰作用量\\)。

    \\(将x^{(0)}(k)+az^{(1)}(k)=b\\) 展开成如下形式：

    $$\left[\matrix{x^{(0)}(2)\\\x^{(0)}(3)\\\x^{(0)}(4)\\\ ...\\\x^{(0)}(n)}\right]=\left[\matrix{-\frac{1}{2}(x^{(1)}(1)+x^{(1)}(2))&1\\\ -\frac{1}{2}(x^{(1)}(2)+x^{(1)}(3))&1\\\ -\frac{1}{2}(x^{(1)}(3)+x^{(1)}(4))&1\\\ ...& ...\\\ -\frac{1}{2}(x^{(1)}(n-1)+x^{(1)}(n))&1}\right]$$
    \\(令Y=\left[\matrix{x^{(0)}(2)\\\x^{(0)}(3)\\\x^{(0)}(4)\\\ ...\\\x^{(0)}(n)}\right],B=\left[\matrix{-\frac{1}{2}(x^{(1)}(1)+x^{(1)}(2))&1\\\ -\frac{1}{2}(x^{(1)}(2)+x^{(1)}(3))&1\\\ -\frac{1}{2}(x^{(1)}(3)+x^{(1)}(4))&1\\\ ...& ...\\\ -\frac{1}{2}(x^{(1)}(n-1)+x^{(1)}(n))&1}\right],\phi=\left[\matrix{a &b}\right]^{\mit T},则上式可以写成Y=B\phi\\)

    由最小平方法可以求出:$$\hat{\phi}=\left[\matrix{\hat{a} & \hat{b}}\right]^{\mit T}=(B^{\mit T}B)^{-1}B^{T}Y$$
    将其代入,求出离散解为:$$\hat{x^{(1)}}(k+1)=(x^{(0)}(1)-\frac{\hat{b}}{\hat{a}})e^{-\hat{a}K}+\frac{\hat{b}}{\hat{a}} \\ \\ \\ \\  \\ \\ \\ \\ \\ \\ (1)$$
    对于原始数据,有：$$\hat{x}^{(0)}(k+1)=\hat{x}^{(1)}(k+1)-\hat{x}^{(1)}(k)=(1-e^{\hat{a}})(x^{(0)}(1)-\frac{\hat{b}}{\hat{a}})e^{-\hat{a}k} \\ \\ \\ \\ \\ \\ \\ \\ (2)$$
    (1)、(2)式称为GM(1,1)模型的时间响应函数模型，是我们计算预测值的基本公式。
    
    对于灰微分方程，若将\\(x^{(0)}(k)\\)的时间k视为连续变量t，则数列\\(x^{(1)}\\)可视为时间的函数，记为\\(x^{(1)}=x^{(1)}(t)\\),并让灰导数对应于导数\\(\frac{dx^{(1)}}{dt}\\),背景值\\(z^{(1)}(k)\\)对应于\\(x^{(1)}(k)\\),则得到的GM(1,1)对应的白微分方程：$$\frac{dx^{(1)}}{dt}+ax^{(1)}=b$$称之为GM(1,1)的白化型。

3. 精度检验

    对于得到的预测值，通常有三种方法进行检验：相对误差大小检验法、后验差检验法、关联度检验法。


 * 相对误差大小检验法

    按GM(1,1)建模法求出\\(\hat{x}^{(1)}\\),并将\\(\hat{X}^{(1)}\\)做一次累减转化为\\(\hat{X}^{(0)}\\),即：$$\hat{X}^{(0)}=[\hat{x}^{(0)}(1),\hat{x}^{(0)}(2),...,\hat{x}^{(0)}(n)]$$
    计算残差：$$E=[e(1),e(2),...,e(n)]=X^{(0)}-\hat{X}^{(0)}$$其中，\\(e(k)=x^{(0)}(k)-\hat{x}^{(0)}(k),k=1,2,...,n\\)
    相对误差：
    $$\cal{rel(k)}=\frac{e(k)}{x^{(0)}(k)}\times100\\%,k=1,2,..,n$$
    平均相对误差：
    $$\cal{rel}=\frac{1}{n}\sum_{k=1}^{n}|\cal{rel(k)}|$$
    如果对于所有\\(\cal{|rel(k)|}<0.1\\),则认为达到较高要求；若对于所有\\(\cal{|rel(k)|}<0.2\\),则认为达到一般要求。

    （这里只介绍相对误差大小检验法，我要去陪女朋友了，后面的那些懒得写了，还有，示例代码中使用的是后验差校验法。）




### 二、代码示例

 根据上面几步，GM的简单Java实现如下：

    /**
    * 灰度模型在Java上的简单实现
    */
    public class GrayModel {
        //na -> -a ,c0 -> b/na ; c1 -> x0_1 - b/na
        private double na, c0, c1;
        //原始数据的大小
        private int size;
        //X0的方差
        private double error;

        public GrayModel(double[] x0) {
            size = x0.length;
            //计算生成数据列 X1
            double[] x1 = new double[size];
            x1[0] = x0[0];
            // avg_x0 x0 的平均值
            int avg_x0=0;
            for (int i = 1; i < size; i++) {
                x1[i] = x0[i] + x1[i - 1];
                avg_x0+=x0[i];
            }
            avg_x0/=size;

            //计算B、BT、Y
            double[][] B = new double[size - 1][2];
            double[][] BT = new double[2][size - 1];
            double[][] Y = new double[size - 1][1];
            for (int i = 0; i < B.length; i++) {
                B[i][0] = -(x1[i] + x1[i + 1]) / 2;
                B[i][1] = 1;
                BT[0][i] = B[i][0];
                BT[1][i] = 1;
                Y[i][0] = x0[i + 1];
            }

            //计算phi
            double[][] phi;
            //这么写只是为了更清晰
            double[][] temp;
            temp= multiply(BT, B);
            temp = multiply(inverse(temp), BT);
            phi = multiply(temp, Y);

            na = -phi[0][0];
            c0 = phi[1][0] / phi[0][0];
            c1 = x0[0] - c0;

            //后验差校验法
            error = 0;
            for (double v : x0) {
                error += Math.sqrt(v - avg_x0);
            }
            error /= size;
        }

        /**
        * 后验差校验法 获取方差
        */
        public double getError() {
            return error;
        }

        /**
        *  获取第k个生成数据
        */
        private double getX1(int k) {
            return c1 * Math.exp(na * k) + c0;
        }

        /**
        * 获取第k个原始数据
        */
        private double getX0(int k) {
            if (k == 0) {
                return c1 * Math.exp(na * k) + c0;
            } else {
                return c1 * (Math.exp(na * k) - Math.exp(na * (k - 1)));
            }
        }

        /**
        * 预测未来的第t个时间点的值
        */
        public double next(int t) {
            assert t > 0 : "输入的index值不能小于0！";
            return getX0(size + t);
        }

        /**
        * 2x2 矩阵求逆
        */
        private static double[][] inverse(double[][] t) {
            double[][] a = new double[2][2];
            double det = t[0][0] * t[1][1] - t[0][1] * t[1][0];
            a[0][0] = t[1][1] / det;
            a[0][1] = -t[1][0] / det;
            a[1][0] = -t[0][1] / det;
            a[1][1] = t[0][0] / det;
            return a;
        }

        /**
        * 简单的矩阵乘法 别问我为什么用两个循环做...
        */
            private static double[][] multiply(double[][] left, double[][] right) {
            int line_left = left.length;
            int row_left = left[0].length;
            int row_right = right[0].length;

            double[][] dest = new double[left.length][right[0].length];
            for (int k = 0; k < line_left; k++) {
                for (int s = 0; s < row_right; s++) {
                    dest[k][s] = 0;
                    for (int i = 0; i < row_left; i++) {
                        dest[k][s] += left[k][i] * right[i][s];
                    }
                }
            }
            return dest;
        }
    }


简单测试一下：

    public static void main(String[] args) {
        double step = 0.001;
        double t = step;
        double[] x0 = new double[10];
        //使用sin+cos 模拟原始数据
        for (int i = 0; i < x0.length; i++) {
            x0[i] = Math.sin(t) + Math.cos(t);
            t += step;
        }
        GrayModel gm = new GrayModel(x0);
        for (int i = 0; i < 10; i++) {
            // 真实值与预测值的差值
            System.out.println(Math.sin(t) + Math.cos(t) - gm.next(i));
            t += step;
        }
        //获取方差
        System.out.println(Math.sqrt(gm.getError()));
    }


### 三、关于使用GM的一点小建议

从GM的计算方式可以看出，GM使用指数曲线来拟合原始数据，所以GM对单调函数具有较为准确的预测，但是对于周期函数，预测的误差则较大。
具体到我接触的代码所设计的业务，几乎没有任何指导意义（简直在扯淡好吗。。。生活中有多少数据的单调递增的），完全是xjb套公式。。。

所以在使用GM的时候，要了解清楚所要分析的数据是否适合使用GM,不要脱离实际,搞花架子忽悠人。




    



    


    







