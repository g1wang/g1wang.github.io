Floyd算法又称为弗洛伊德算法，插点法，是一种用于寻找给定的加权图中顶点间最短路径的算法。

算法思想

通过一个图的权值矩阵求出它的每两点间的最短路径矩阵。
从图的带权邻接矩阵A=[a(i,j)] n×n开始，递归地进行n次更新，即由矩阵D(0)=A，动态规划得到D(n)。矩阵D(n)的i行j列元素便是i号顶点到j号顶点的最短路径长度，称D(n)为图的距离矩阵，同时还可引入一个后继节点矩阵path来记录两点间的最短路径。

算法描述
　a)　初始化：
```
D[u,v]=A[u,v]　
```
　b)　
```
　For k:=1 to n
　　　　For i:=1 to n
　　　　　For j:=1 to n
　　　　　　If D[i,j]>D[i,k]+D[k,j] Then
　　　　　　　D[i,j]:=D[i,k]+D[k,j];
```

　c)　算法结束：D即为所有点对的最短路径矩阵

时间复杂度

O(n^3)

优缺点分析
Floyd算法适用于APSP(All Pairs Shortest Paths)，稠密图效果最佳，边权可正可负。此算法简单有效，由于三重循环结构紧凑，对于稠密图，效率要高于执行|V|次Dijkstra算法。
优点：容易理解，可以算出任意两个节点之间的最短距离，代码编写简单；
缺点：时间复杂度比较高，不适合计算大量数据。
```
/**
* Floyd
*
* @author CodeX
* @version 1.0
* @date 2021/4/27 12:18
*/

public class Floyd {

    public static void main(String[] args) {
        int max = Integer.MAX_VALUE;
        int[][] weights = new int[][]{
                {max, max, 10, max, 30, 100},
                {max, max, 5, max, max, max},
                {max, max, max, 50, max, max},
                {max, max, max, max, max, 10},
                {max, max, max, 20, max, 60},
                {max, max, max, max, max, max}};

        int[][] dp = floyd(weights);
        for (int i = 0; i < dp.length; i++) {
            StringBuffer sb = new StringBuffer();
            for (int j = 0; j < dp.length; j++) {
                sb.append(dp[i][j] + " ");
            }
            System.out.println(sb.toString().trim());
        }

    }

    private static int[][] floyd(int[][] weights) {
        int[][] dp = weights;
        int len = dp.length;
        for (int i = 0; i < len; i++) {
            for (int j = 0; j < len; j++) {
                for (int k = 0; k < len; k++) {
                    if (dp[j][i] != Integer.MAX_VALUE && dp[i][k] != Integer.MAX_VALUE && dp[j][k] > dp[j][i] + dp[i][k]) {
                        dp[j][k] = dp[j][i] + dp[i][k];
                    }
                }
            }
        }
        return dp;
    }
}
```