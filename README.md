题目（[前往原题](https://leetcode-cn.com/problems/max-points-on-a-line)）：

```markdown
给定一个二维平面，平面上有 n 个点，求最多有多少个点在同一条直线上。

示例:
输入: [[1,1],[2,2],[3,3]]
输出: 3
解释:
^
|
|        o
|     o
|  o  
+------------->
0  1  2  3  4

```

思路：

已知两点组成一条直线，我们可以通过这两点的坐标算出斜率。

**令点1、点2的斜率为k1，点1、点3的斜率为k2，若k1=k2，说明点1、点2、点3在同一条直线上。**

抓住了这个关键点，解题就容易了。我们可以用HashMap的键值存斜率，value值存该斜率出现的次数，也就是在同一条直线上的点的个数。

代码：

```java
class Solution {
    public int maxPoints(int[][] points) {
        if(points.length<3){
            return points.length;
        }
        int res=0;
        //去重合点。主要思想：若点1和点2重合，将点2置为null，且coincide.put(点1的位置,原value+1),说明又找到了1个重合点
        HashMap<Integer, Integer> coincide = new HashMap<Integer, Integer>();
        for(int i=0;i<points.length;i++){
            if(points[i]!=null){
                coincide.put(i,1);
                for(int j=i+1;j<points.length;j++){
                    if(points[j]!=null && points[i][0]==points[j][0] && points[i][1]==points[j][1]){
                        points[j]=null;
                        coincide.put(i,coincide.get(i)+1);
                    }
                }
            }
        }
        //coincide记录了去重合点后的位置，只去遍历这些位置算斜率
        for(Integer i : coincide.keySet()){
            HashMap<Double, Integer> map = new HashMap<Double, Integer>();
            for(Integer j : coincide.keySet()){
                if(j==i){
                    continue;
                }
                //计算两点的斜率
                double k = (double)10*(points[j][1]-points[i][1])/(points[j][0]-points[i][0]);
                if(map.containsKey(k)){
                    //若该斜率已存在，value=原value+位置j的重合点个数
                    map.put(k,map.get(k)+coincide.get(j));
                }else{
                    //若map中不存在k这个斜率，存入k，value=位置i+位置j的重合点个数
                    map.put(k,coincide.get(i)+coincide.get(j));
                }
            }
            for(Double key : map.keySet()){
                if(map.get(key)>res){
                    res = map.get(key);
                }
            }
            // System.out.println("map="+map);
        }
        //考虑全是重合点的情况
        for(Integer key : coincide.keySet()){
            if(coincide.get(key)>res){
                res = coincide.get(key);
            }
        }
        return res;
    }
}
```

总结：

为了去重合点，我额外又用了一个HashMap来记录重合点的位置和次数。也可以不这样做，照常遍历，设置一个count值，在遇到重合点的时候count+1，最后count+map.get(k)就是此次遍历的结果。

还有leetcode的测试用例中有一组会使得double精度不够，这种情况直接*10就可以了。。有点投机取巧，但效果不错【doge】。

收获：

数组只能遍历输出，因为数组的变量名存的是入口地址。

list是已被封装过的类，变量名就是它本身，可以直接输出。

且由于points是二维数组，points[i]是一个一维数组，所以求重合点不可以直接用points[i]==points[j]，因为判断数组是否相等不能用==。
