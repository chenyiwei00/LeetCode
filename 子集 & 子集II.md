子集题：

![img](https://pic1.zhimg.com/v2-17a639bf581b95a3f2e917e516ddce68_b.png)

思路：

求子集问题，很自然想到用回溯法。

代码：

```java
class Solution {
    List<List<Integer>> res = new ArrayList();
    List<Integer> temp = new ArrayList();
    public void backtrack(int[] nums,int k){
        for(int i=k;i<nums.length;i++){
            temp.add(nums[i]);
            //另外声明一个A列表，将temp的内容拷贝至A列表，再将A列表加进res，从而实现深拷贝
            List<Integer> A = new ArrayList();
            A.addAll(temp);
            res.add(A);
            backtrack(nums,i+1);
            //删除temp列表的最后一个元素
            temp.remove(temp.size()-1);
        }
    }
    public List<List<Integer>> subsets(int[] nums) {
        backtrack(nums,0);
        //回溯结束后，加上空集
        List<Integer> A = new ArrayList();
        res.add(A);
        return res;
    }
}
```

总结：

由于nums是不包含重复元素的，所以我们用常规的回溯法就可以得出结果。

这题的关键是要理解**Java的深拷贝和浅拷贝**。

浅拷贝：对于**引用数据类型**，拷贝的是它的引用，并没有创建一个新的对象，即没有分配新的内存空间。（对于基本数据类型，拷贝的就是它的值）

所以，若我们使用res.add(temp)，后面再对temp进行操作，原先加进res里的那个temp会一直随着原temp后续的改变而改变。

而深拷贝就是在引用类型进行拷贝时，创建了新的对象，即分配了新的内存空间给拷贝对象。

子集 II：

![img](https://pic1.zhimg.com/v2-f14f7048b4e08ea14e008e51c28701d0_b.png)

思路：此题将nums改成了可能包含重复元素，所以，如果是常规的遍历，势必会取到重复子集。为了避免这种情况，我在每次add前，先判断res列表中是否已包含该子集。

代码：

```java
class Solution {
    List<List<Integer>> res = new ArrayList();
    List<Integer> temp = new ArrayList();
    public void backtrack(int[] nums,int k){
        for(int i=k;i<nums.length;i++){
            temp.add(nums[i]);
            //add前先判断res里是否已包含该子集。依然要用深拷贝
            if(!res.contains(temp)){
                List<Integer> A = new ArrayList();
                A.addAll(temp);
                res.add(A);
            }
            backtrack(nums,i+1);
            temp.remove(temp.size()-1);
        }
    }
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums);
        backtrack(nums,0);
        List<Integer> A = new ArrayList();
        res.add(A);
        return res;
    }
}
```

总结：重复元素的问题不难解决，这里有个细节是要在回溯前将nums数组进行排序，这样才能保证用contains()方法能正确判断出重复子集。若不排序，两个重复子集的内部排列可能是不同的，list会认为是不同的元素。

数组排序可用Arrays.sort(nums);

注意：Arrays类只提供默认的升序排列（对本题没有影响）。
