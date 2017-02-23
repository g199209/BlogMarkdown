title: N皇后问题
permalink: N_Queen_Problem
toc: false
mathjax: false
fancybox: false
tags: [算法]
categories: 编程
date: 2016-11-22 22:39:36

---

N皇后问题是经典八皇后问题的扩展：在N*N的棋盘上，有N个皇后需要放置，需满足任意两个皇后不能位于同一行、同一列或者是同一对角线上，求一共有几种放置方法。

<!--more-->

N皇后问题是一个经典的回溯法的例子，核心思想就是逐行（或逐列）放置，若某一行没有可供放置的位置了，说明前面的放置有误，故回溯到上一行寻找下一个可能的位置。N皇后问题是一个NP-Hard问题，其算法复杂度是指数复杂度的。下面给出两种实现方法。

## 经典回溯非递归实现

``` C
int NQueens(int N) {
    int * columns;
    int i, j, k, result;

    // 初始化
    columns = (int *)malloc(sizeof(int) * N);
    result = 0;
    for (i = 0; i < N; i++) {
        columns[i] = -1;  // -1代表还未放置
    }

    for (i = 0; i >= 0;) {
        // 寻找第i列下一个可供放置的位置
        for (j = columns[i] + 1; j < N; j++) {
            // 检查位置是否冲突
            for (k = 1; i - k >= 0; k++) {
                if (columns[i - k] == j || columns[i - k] == j - k || columns[i - k] == j + k) {
                    break;
                }
            }
            // 有位置可用
            if (i - k < 0) {
                columns[i] = j;
                break;
            }        
        }

        // 第i列已经没有空位，需要回溯
        if (j == N) {
            columns[i] = -1;
            i--;
            continue;
        }
        // 到达最后一列，找到一个解
        if (i == N-1){
            result++;
        }
        // 继续寻找下一列
        else {
            i++;
        }
    }
    free(columns);
    return result;
}
```

程序很简单，就不多说明了。

## 位运算递归实现
上面使用数组实现的回溯法其实效率很低，N皇后最高效的解法是位运算算法，下面给出实现代码：

``` C
static int upperlimit;
static int count;
int main(){
    int N;
    printf("Enter N : ");
    scanf("%d", &N);
    count = 0;
    upperlimit = (1 << N) - 1;
    NQueen(0,0,0);
    printf("Solution : %d\n", count);
}
 
void NQueen(int row,int ld,int rd){
    int pos, p;
     
    //row == upperlim,说明皇后全部放进去了 
    if(row != upperlimit){
         
        //row，ld，rd进行“或”运算，如：000101 ，1的列表示已经放置了皇后
        //pos 取反得 111010，1表示可以放皇后的列  
        //upperlimit 是上限值,控制二进制长度:
        //如：(~(row | ld | rd ) = 100001011101时,upperlimit = 111111,pos= 011101 
        pos = upperlimit & (~(row | ld | rd ));  
        while (pos != 0) {
             
            // 找出可以放皇后的位置（默认从右到左），如：p=000100,表示该行右边第三个位置可以放皇后。
            // p就表示该行的某个可以放皇后的位置，把皇后放在这个位置上后，就把它从pos中移除并递归调用test过程。
            p = pos & (~pos + 1);  
             
            // 把皇后放在这个位置上后，就把它从pos中移除并递归调用test过程。
            // 如：pos = 111100时,p = 000100,把皇后放在这个位置上后， pos = pos - p, pos=111000
            pos = pos - p;  
            //(row|p) 是计算已经在对应列上面的皇后
            //(ld | p)<< 1 是因为由ld造成的占位在下一行要右移一下；
            // (rd | p)>> 1 是因为由rd造成的占位在下一行要左移一下
            test((row|p),(ld|p)<<1,(rd|p)>>1);
        }
    }else {
        count++;
    }
}
```

此算法的详细解释见：

> [八皇后问题详解及代码实现(位运算算法)](https://my.oschina.net/CodingMen/blog/715983)
> [N皇后问题的两个最高效的算法](http://blog.csdn.net/hackbuteer1/article/details/6657109)
> [位运算简介及实用技巧（三）：进阶篇(2)](http://www.matrix67.com/blog/archives/266)

实际测试表明，此方法的效率是简单回溯的10倍左右。

----------

附注：1~27皇后问题的解法数量见：[A000170](https://oeis.org/A000170/list)，这个可以用来验证自己写的算法是否正确。
