### 分享我的LeetCode刷题记录^__^
我会根据不同的解法对题目进行分类，知乎专栏也会同步更新~[我的知乎专栏](https://zhuanlan.zhihu.com/c_1187843464115523584)

```markdown
import java.util.Scanner;

public class Main {
    private static int backtrack(Integer[][] dp, int[] nums, int n, int B){
        //若dp[n][B]已经算过，则返回
        if(dp[n][B] != null)
            return dp[n][B];
        int min = Integer.MAX_VALUE;
        for(int i=1;i<B+1;i++){
            if(B%i != 0)
                continue;
            //找出B的一对因子i和j
            int j = B/i;
            //求出用B的多对因子求代价的最小值
            min = Math.min(min,backtrack(dp,nums,n-1,i)+Math.abs(nums[n-1]-j));
        }
        return dp[n][B] = min;
    }
    public static void main(String[] args){
       Scanner scan  = new Scanner(System.in);
       int n = scan.nextInt();
       int B = scan.nextInt();
       int[] nums = new int[n];
       for(int i=0;i<n;i++)
           nums[i] = scan.nextInt();
       Integer[][] dp = new Integer[n+1][B+1];
       //为二维数组第一行与第一列赋值（其实是第二行第二列，这里为了便于理解不去管dp[0][j]和dp[i][0]的情况）
       for(int i=1;i<B+1;i++)
           dp[1][i] = Math.abs(i-nums[0]);
       int temp = 0;
       //第一列相应位置的值=令前i个数都变为1的代价之和
       for(int i=1;i<n+1;i++){
           temp += Math.abs(nums[i-1]-1);
           dp[i][1] = temp;
       }
       System.out.println(backtrack(dp,nums,n,B));
    }
}
```
