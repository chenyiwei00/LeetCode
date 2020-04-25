题目（[前往原题](https://leetcode-cn.com/problems/repeated-dna-sequences)）：

```markdown
所有 DNA 都由一系列缩写为 A，C，G 和 T 的核苷酸组成，例如：“ACGAATTCCG”。在研究 DNA 时，识别 DNA 中的重复序列有时会对研究非常有帮助。
编写一个函数来查找 DNA 分子中所有出现超过一次的 10 个字母长的序列（子串）。

示例：
输入：s = "AAAAACCCCCAAAAACCCCCCAAAAAGGGTTT"
输出：["AAAAACCCCC", "CCCCCAAAAA"]
```

思路：这道题如果用常规的双重循环查找子串会超出时间限制，所以考虑一次遍历解决问题。

解法一（HashSet）：

代码：

```java
class Solution {
    public List<String> findRepeatedDnaSequences(String s) {
        //用HashSet保证不会添加重复值
        Set<String> res = new LinkedHashSet<>();
        Set<String> temp = new LinkedHashSet<>();
        //截取长度为10的子串，若temp中不存在该子串，则添加进temp，若已存在，则说明该子串有重复子串，添加进res
        for(int i=0;i<s.length()-9;i++){
            String key = s.substring(i,i+10);
            if(temp.contains(key)){
                res.add(key);
            }else{
                temp.add(key);
            }
        }
        //return前将Set转化为List
        return new ArrayList<>(res);
    }
}
```

总结：利用hashSet不会添加重复元素的特性，来完成去重。

这里记录一下List、Set、Array相互转化的方法：

**Array转List：**

String[] arr = new String[n];

List<String> list = Arrays.asList(arr);（注意，若后续修改arr中的元素，list中相对应的值也会随之改变）

**List转Array：**

List<String> list = new ArrayList();

String[] arr = list.toArray(newString[list.size()]);（使用toArray方法，若后续修改list中的元素，arr中相对应的值不会随之改变）

**List转Set：**

List<String> list = new ArrayList();

Set<String> set = new HashSet<>(list);

**Set转List：**

Set<String> set = new HashSet<>();

List<String> list = new ArrayList<>(set);

这两种转化和toArray一样，若后续修改list(set)的值，set(list)中相当于的值不会改变。

Set和Array的相互转化，可综合上面的几种方法进行处理。

解法二（HashMap）：

```java
class Solution {
    public List<String> findRepeatedDnaSequences(String s) {
        HashMap<String, Integer> map = new HashMap<String, Integer>();
        List<String> res = new ArrayList();
        for(int i=0;i<s.length()-9;i++){
            String key = s.substring(i,i+10);
            //判断map中是否存在该key，若存在，将key值加进res，并将map中相对应的value值置为2（去重）
            if(map.containsKey(key) && map.get(key)==1){
                res.add(key);
                map.put(key,2);
            }else if(!map.containsKey(key)){
                map.put(key,1);
            }
        }
        return res;
    }
}
```

总结：

这两种解法的思想是一样的，只是在去重的处理细节上不一样。

只是使用HashMap的执行起来会比HashSet快，查了下资料，是因为HashMap使用唯一的键来获取对象，所以查询速度比HashMap要快一些。
