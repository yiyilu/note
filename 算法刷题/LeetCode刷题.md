# HOT100

## 3.无重复字符的最长子串

> 给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长子串** 的长度。
>
> 方法：暴力或滑动窗口；
>
> 滑动窗口：如abcabcbb这个字符串，把他当做一个队列，一开始进去的是abc，后面a 进来的话发现有重复字符，所以第一个a要出队列，然后把边界移动。 字符可以用哈希表存，键是字符，值是字符对应的下标。

```Java
//暴力做法
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int res=0;
        int count=0;
        List<Character> listCh = new ArrayList<>();
        int cursor=0;
        for(int i=0;i<s.length();i++){
            if(!listCh.contains(s.charAt(i))){
                listCh.add(s.charAt(i));
                count++;
            }             
            else{
                res= Math.max(res,count);
                count=0;
                listCh.removeAll(listCh);  
                i=cursor;
                cursor++;
            }
        }
        res=Math.max(res,count);
        return res;
    }
}
```



```Java
//滑动窗口
class Solution {
    public int lengthOfLongestSubstring(String s) {
        if(s.length()==0)
        return 0;
        Map<Character,Integer> map = new HashMap<>();
        //left是用来记录边界的下表，初始是为0，
        int left = 0;
        int res=0;
        for(int i=0;i<s.length();i++){
            //如果字符在哈希表中存在的话，更新边界
            if(map.containsKey(s.charAt(i))){
                //为什么在这边要用max，而不是直接left= map.get(s.charAt(i)+1),是因为比如abba,当第二个b进来的时候，此时left=0,map状态为(a,0),(b,1), 因为重复了，所以left更新为2，map更新为(a,0),(b,2)，当第二个a进来的时候，此时map为(a,0),(b,2),left=2,因为重复了，所以要更新left，假如是用left直接等于map.get(s.charAt(i))+1的话，得到的left是1，原来的边界是2，那边界就又回去了，所以要跟原有的left比较得出大的那一个。
                
                left = Math.max(left,map.get(s.charAt(i))+1);
                
            }
            
            map.put(s.charAt(i),i);
            //字段长度为i-left+1，更新最大值
            res= Math.max(res,i-left+1);
        }
        return res;
    }
}
```



## 64.最小路径和

> 给定一个包含非负整数的 `*m* x *n*` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。
>
> **说明：**每次只能向下或者向右移动一步。
>
> 做法：这属于一个动态规划的题目，移动的时候肯定是从第一行或第一列开始向右向下移动，所以我们先算出在第一列、第一行上的dp情况，中间的部分循环比较，比较上面的和左边的dp值哪个小就加哪个，最后到右下角的那个就是最小的dp值了。
>
> 看的题解

```Java
class Solution {
    public int minPathSum(int[][] grid) {

        if(grid==null|| grid[0].length==0|| grid.length==0){
            return 0;
        }
        
        int rows= grid.length;
        int columns = grid[0].length;
        int[][] dp = new int[rows][columns];
        dp[0][0]= grid[0][0];
        for(int i=1;i<rows;i++){
            dp[i][0]=dp[i-1][0]+grid[i][0];
        }

        for(int j=1;j<columns;j++){
            dp[0][j]=dp[0][j-1]+grid[0][j];
        }
        for(int i=1;i<rows;i++){
            for(int j=1;j<columns;j++){
                dp[i][j]= Math.min(dp[i-1][j], dp[i][j-1])+grid[i][j];
            }
        }

        return dp[rows-1][columns-1];

    }
}
```





## 78.子集

> 给你一个整数数组 `nums` ，数组中的元素 **互不相同** 。返回该数组所有可能的子集（幂集）。
>
> 解集 **不能** 包含重复的子集。你可以按 **任意顺序** 返回解集。
>
> 思路：遍历数组元素的时候，用一个all来计算当前res的长度，比如说示例【1,2,3】，遍历1的时候，res中有一个空List，长度为1， 然后新建一个list，在那个空List的基础上add nums[i], 再把这个新的list加入res。
>
> 看的题解

```Java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> res = new ArrayList<>();
        res.add(new ArrayList<>());
        for(int i=0;i<nums.length;i++){
            int all = res.size();
            for(int j=0;j<all;j++){
//这边不能写List<Integer> tmp = res.get(j)，因为tmp指向的仍然是res中j的地址，当tmp改变了，res.get(j)也会改变，所以虽然res.add(tmp)了，但是之前res.get(j)的值也变了，会导致结果混乱。
                List<Integer> tmp = new ArrayList<>(res.get(j));
                tmp.add(nums[i]);
                res.add(tmp);
            }
        }
        return res ;

    }
}
```



## 152.乘积最大子数组

> 给你一个整数数组 `nums` ，请你找出数组中乘积最大的连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。
>
> 思路：{5,6,−3,4,−3}， 对于这个数组，如果只是按一般的动态规划做的话，即比较当前位置，和当前位置与上一个位置的maxDP乘积相比谁更大，出来的结果就是{5,30，-3,4，-3}， 拿到的最大值是30，但实际上最大的是5 * 30 *（-3） *4 *（-3），所以得考虑负数的情况。对于当前位置是负数的情况，肯定是希望上一个位置的dp值也是负数，而且越小越好。可以额外new一个minDP,用来记录每个位置上的最小乘积。
>
> 看的题解

```Java
class Solution {
    public int maxProduct(int[] nums) {
       int length = nums.length;
       int[] maxDp = new int[length];
       int[] minDp = new int[length];
       System.arraycopy(nums,0,maxDp,0,length);
       System.arraycopy(nums,0,minDp,0,length);
       for(int i=1;i<length;i++){
           maxDp[i] =Math.max(Math.max(maxDp[i-1]*nums[i], minDp[i-1]*nums[i]), nums[i]);
           minDp[i] =Math.min(Math.min(maxDp[i-1]*nums[i], minDp[i-1]*nums[i]), nums[i]);        
       }
       int res=maxDp[0];
       for(int i=1;i<length;i++){
           res= Math.max(res,maxDp[i]);
       }
       return res;

    }
}
```



## 160.相交链表

> 给你两个单链表的头节点 headA 和 headB ，请你找出并返回两个单链表相交的起始节点。如果两个链表没有交点，返回 null 。
>
> 图示两个链表在节点 c1 开始相交：
>
> 比如说headA = 1->2->3->4
>
> headB = 5->1->2->3->4
>
> 他们在2处相交，至于为什么不是在1处相交，题目的意思应该不只是node的val相等，而是node的地址相等，这样才算严格意义上的相交。

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if(headA==null || headB==null)
        return null;
        ListNode p1= headA;
        ListNode p2= headB;
        //什么时候跳出循环？ 当p1走了M+N步，p2也走了M+N步之后，两个肯定都是null值，此时就是跳出循环了
        while(p1!=p2){
            p1= p1==null?headB:p1.next;
            p2=p2==null?headA:p2.next;
        }
        return p1;
        
    }
}
```



## 560.和为K的子数组

> 给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回该数组中和为 `k` 的连续子数组的个数。

第一遍自己做的，暴力枚举

```Java
class Solution {
    public int subarraySum(int[] nums, int k) {
        int res=0;
        for(int i=0;i<nums.length;i++){
            int sum =nums[i];
            if(sum==k){
                res++;
            }
            for(int j=i+1;j<nums.length;j++){
                sum+=nums[j];
                if(sum==k){
                    res++;
                }      
            }
        }
        return res;

    }
}
```

题解：计算每个nums[i]对应位置上的和，比如说【1,1,2】 前缀和就是【1,2,4】， 前缀和等于nums[i]+pre。 这里有个规律，比如要有k为3的连续子数组，前缀和中肯定有（某个nums[i]的pre值-3 ） 这个数，就比如说前缀和中4-3=1 ，4和1都存在前缀和数组里面，所以我们只要判断pre-k是否存在

```Java
class Solution{
    public int subarraySum(int[] nums, int k){
        Map<Integer, Integer> map = new HashMap<>();
        map.put(0,1);
        int pre=0;
        int count=0;
        for(int i=0;i<nums.length;i++){
            pre+=nums[i];
            if(map.containsKey(pre-k)){
                count+=map.get(pre-k);
            }
            map.put(pre, map.getOrDefault(pre,0)+1);
        }

        return count;

    }
}
```



## 200.岛屿数量

> 给你一个由 '1'（陆地）和 '0'（水）组成的的二维网格，请你计算网格中岛屿的数量。
>
> 岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。
>
> 此外，你可以假设该网格的四条边均被水包围。
>
> 输入：grid = [
>   ["1","1","1","1","0"],
>   ["1","1","0","1","0"],
>   ["1","1","0","0","0"],
>   ["0","0","0","0","0"]
> ]
> 输出：1
>
> 思路：其实就是要你求最多联通块的数量，1代表联通

1. DFS求最大连通块

   ```Java
   class Solution {
       public void dfs(char[][] grid, int i, int j){
           if(i<0||i>=grid.length||j<0||j>=grid[0].length||grid[i][j]=='0')
           return;
           //把查询过的设置为0,因为已经用过了
           grid[i][j]='0';
           //上面
           dfs(grid,i-1,j);
           //左面
           dfs(grid,i,j-1);
           //右面
           dfs(grid,i,j+1);
           //下面
           dfs(grid,i+1,j);
       }
       public int numIslands(char[][] grid) {
           if(grid==null||grid.length==0||grid[0].length==0){
               return 0;
           }
           int num=0;
           for(int i=0;i<grid.length;i++){
               for(int j=0;j<grid[0].length;j++){
                   if(grid[i][j]=='1'){
                       num++;
                       dfs(grid,i,j);
                   }
               }
           }
           return num;
       }
   }
   ```

2. DFS求最大连接1的数量

   ```java
   class Solution {
       public int dfs(char[][] grid, int i, int j,int sum){
          if(i<0||i>=grid.length||j<0||j>=grid[0].length||grid[i][j]=='0')
           return 0;
           else 
           sum+=1;
           grid[i][j]='0';
           //上面
           sum+=dfs(grid,i-1,j,0);
           //左面
           sum+=dfs(grid,i,j-1,0);
           //右面
           sum+=dfs(grid,i,j+1,0);
           //下面
           sum+=dfs(grid,i+1,j,0);
   
           return sum;
       }
       public int numIslands(char[][] grid) {
           if(grid==null||grid.length==0||grid[0].length==0){
               return 0;
           }
           int max=0;
           for(int i=0;i<grid.length;i++){
               for(int j=0;j<grid[0].length;j++){
                   if(grid[i][j]=='1'){       
                       int dfs = dfs(grid,i,j,0);             
                       max=Math.max(dfs,max);
                       //if(dfs>0)
                        //System.out.println(dfs);
                       
                   }
               }
           }
           return max;
       }
   }
   ```

   





# 摩尔投票

## 169 多数元素

> 给定一个大小为 n 的数组，找到其中的多数元素。多数元素是指在数组中出现次数 大于 ⌊ n/2 ⌋ 的元素。
>
> 你可以假设数组是非空的，并且给定的数组总是存在多数元素。
>

我是用hashmap做的，记录每一个数字出现的次数，超过半数则返回。

题解中说可以用摩尔投票做，思路：题目中说的多数元素指在数组中出现次数 大于 ⌊ n/2 ⌋ 的元素，那么其余元素的和肯定是小于等于 ⌊ n/2 ⌋ ，比如说【3,2,3】， 3>1,所以3是多数元素， 2<=1。 [2,2,1,1,1,2,2]，2>3,2是多数元素。总之，多数元素-其余元素的和肯定是>=1的。摩尔投票中，将候选人初始化为nums[0], 次数count初始化为1， 遇到相同则+1，遇到不同则-1，如果count=0了则更换候选人，重置count=1; 多数元素和其他元素两两抵消，最后的剩余肯定是>=1，所以行得通。

```java
        //摩尔投票
        int candidate = nums[0];
        int count=1;
        for(int i=1;i<nums.length;i++){
            if(candidate==nums[i])
            count++;
            else
            count--;
            if(count==0){
                candidate=nums[i];
                count++;
            }
        }
        return candidate;
```



## 229 求众数2

给定一个大小为 n 的整数数组，找出其中所有出现超过 ⌊ n/3 ⌋ 次的元素。

进阶：尝试设计时间复杂度为 O(n)、空间复杂度为 O(1)的算法解决此问题。



```java
class Solution {
    public List<Integer> majorityElement(int[] nums) {
        // 创建返回值
        List<Integer> res = new ArrayList<>();
        if (nums == null || nums.length == 0) return res;
        // 初始化两个候选人candidate，和他们的计票
        //为什么有两个候选人？是因为超过n/3的元素最多有2个，2个超过n/3, 其余的肯定是小于n/3的了。不可能有3个元素都超过n/3的。
        int cand1 = nums[0], count1 = 0;
        int cand2 = nums[0], count2 = 0;

        // 摩尔投票法，分为两个阶段：配对阶段和计数阶段
        // 配对阶段
        for (int num : nums) {
            // 投票
            if (cand1 == num) {
                count1++;
                continue;
            }
            if (cand2 == num) {
                count2++;
                continue;
            }
           

            // 第1个候选人配对
            if (count1 == 0) {
                cand1 = num;
                count1++;
                continue;
            }
            // 第2个候选人配对
            if (count2 == 0) {
                cand2 = num;
                count2++;
                continue;
            }

            //为什么这边两个count都要-1,是因为这两个候选人是一个队伍的，要败一起败，所以都要-1。关于这个阵营，可以看下面别人的解释。
            count1--;
            count2--;
        }

        // 计数阶段
        // 找到了两个候选人之后，需要确定票数是否满足大于 N/3
        count1 = 0;
        count2 = 0;
        for (int num : nums) {
            if (cand1 == num) count1++;
            else if (cand2 == num) count2++;
        }

        if (count1 > nums.length / 3) res.add(cand1);
        if (count2 > nums.length / 3) res.add(cand2);

        return res;
    }
}
```

遍历了两次，时间复杂度o(2n), 也就是o(n),相当于线性的，空间复杂度o(1),因为只开辟了常量空间。

梳理:

> 如果最多选1个代表，那么它的票数大于n/2;
>
> 如果最多选2个代表，那么它的票数大于n/3;
>
> 以此类推，如果最多选m个代表，那么它的票数要大于n/(m+1)

别人对摩尔投票的比喻

> 有一个对摩尔投票法非常形象的比喻：多方混战。
>
> 首先要知道，在任何数组中，出现次数大于该数组长度1/3的值最多只有两个。
>
> 我们把这道题比作一场多方混战，战斗结果一定只有最多两个阵营幸存，其他阵营被歼灭。数组中的数字即代表某士兵所在的阵营。
>
> 我们维护两个潜在幸存阵营A和B。我们遍历数组，如果遇到了属于A或者属于B的士兵，则把士兵加入A或B队伍中，该队伍人数加一。继续遍历。
>
> 如果遇到了一个士兵既不属于A阵营，也不属于B阵营，这时有两种情况：
>
> 1. A阵营和B阵营都还有活着的士兵，那么进行一次厮杀，参与厮杀的三个士兵全部阵亡：A阵营的一个士兵阵亡，B阵营的一个士兵阵亡，这个不知道从哪个阵营来的士兵也阵亡。继续遍历。
> 2. A阵营或B阵营已经没有士兵了。这个阵营暂时从地球上消失了。那么把当前遍历到的新士兵算作新的潜在幸存阵营，这个新阵营只有他一个人。继续遍历。
>
> 大战结束，最后A和B阵营就是初始人数最多的阵营。判断一下A，B的人数是否超过所有人数的三分之一就行了。
>
> 12踩收起回复回复分享举报

​        



# 滑动窗口

## 3.无重复字符的最长子串

> 给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长子串** 的长度。
>
> 方法：暴力或滑动窗口；
>
> 滑动窗口：如abcabcbb这个字符串，把他当做一个队列，一开始进去的是abc，后面a 进来的话发现有重复字符，所以第一个a要出队列，然后把边界移动。 字符可以用哈希表存，键是字符，值是字符对应的下标。

```Java
//暴力做法
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int res=0;
        int count=0;
        List<Character> listCh = new ArrayList<>();
        int cursor=0;
        for(int i=0;i<s.length();i++){
            if(!listCh.contains(s.charAt(i))){
                listCh.add(s.charAt(i));
                count++;
            }             
            else{
                res= Math.max(res,count);
                count=0;
                listCh.removeAll(listCh);  
                i=cursor;
                cursor++;
            }
        }
        res=Math.max(res,count);
        return res;
    }
}
```



```Java
//滑动窗口
class Solution {
    public int lengthOfLongestSubstring(String s) {
        if(s.length()==0)
        return 0;
        Map<Character,Integer> map = new HashMap<>();
        //left是用来记录边界的下表，初始是为0，
        int left = 0;
        int res=0;
        for(int i=0;i<s.length();i++){
            //如果字符在哈希表中存在的话，更新边界
            if(map.containsKey(s.charAt(i))){
                //为什么在这边要用max，而不是直接left= map.get(s.charAt(i)+1),是因为比如abba,当第二个b进来的时候，此时left=0,map状态为(a,0),(b,1), 因为重复了，所以left更新为2，map更新为(a,0),(b,2)，当第二个a进来的时候，此时map为(a,0),(b,2),left=2,因为重复了，所以要更新left，假如是用left直接等于map.get(s.charAt(i))+1的话，得到的left是1，原来的边界是2，那边界就又回去了，所以要跟原有的left比较得出大的那一个。
                
                left = Math.max(left,map.get(s.charAt(i))+1);
                
            }
            
            map.put(s.charAt(i),i);
            //字段长度为i-left+1，更新最大值
            res= Math.max(res,i-left+1);
        }
        return res;
    }
}
```



## 159.至多包含两个不同字符的最长子串

> 给定一个字符串 ***s\*** ，找出 **至多** 包含两个不同字符的最长子串 ***t\*** ，并返回该子串的长度
>
> ```
> 示例：
> 输入: "eceba"
> 输出: 3
> 解释: t 是 "ece"，长度为3。
> ```
>
> 思路：滑动窗口， 窗口最大容量为2， 用一个哈希表存储字符为键，该字符最最右边的位置下表为值。
>
> 当哈希表的size不为3的时候，可以继续往里面Put，当size为3的时候，需要从hashmap中remove掉一个，remove的是最右指针最小的那一个字符，然后left指针指向最右指针最小的那个字符的下标+1；
>
> 拿leeeeeeeetcode为例，一开始left=0,right=0,都指向l,l的最右位置为0，然后right++, 指向e，继续添加，更新e的最右位置到8，然后添加t，最右位置为9,此时哈希表容量为3了，需要remove掉，通过collection.min(map.value()),找到最小的指针，及其对应的字符，在这里最小的应该是l,然后remove掉l,继续添加新字符。。。

```Java
class Solution {
    public int lengthOfLongestSubstringTwoDistinct(String s) {
        int n = s.length();
        if(s.length()<3)
        return n;
        int left =0,right=0;
        int max_len=2;
        Map<Character,Integer> map = new HashMap<>();

        while(right<n){
            if(map.size()<3){
                //这边已经right++了，所以后面求长度的时候只需要right-left。
                map.put(s.charAt(right),right++);
            }
            //哈希表等于3了，需要remove掉一个
             if(map.size()==3){  
                 //找到最左边且值最小的那个字符的下标
                int leftMostIdx = Collections.min(map.values());
                char ch = s.charAt(leftMostIdx);
                 //remove掉这个字符
                map.remove(ch);
                 //left指针更新
                left =leftMostIdx+1;
            }

            max_len = Math.max(max_len,right-left); 
            
        }

        return max_len;

    }
}
```





# 字典序

## 386. 字典序排序

> 给你一个整数 `n` ，按字典序返回范围 `[1, n]` 内所有整数。
>
> 你必须设计一个时间复杂度为 `O(n)` 且使用 `O(1)` 额外空间的算法。
>
> 思路，把数字想象成一颗10叉树，然后用dfs深度遍历，先序输出。 注意第一个数字不为0就行了。

<img src="C:\Users\10293\AppData\Roaming\Typora\typora-user-images\image-20211105105409668.png" alt="image-20211105105409668" style="zoom:50%;" />

迭代法：

```java
class Solution {
    public List<Integer> lexicalOrder(int n) {

        List<Integer> res = new ArrayList<>();
        int num =1;
        while(res.size()<n){
            while(num<=n){
                res.add(num);
                num=num*10;
            }
            while(num%10==9 || num>n){
                num=num/10;
            }
            num+=1;
        }
        return res;
    }
}
```

递归法

```java
class Solution{
    public void dfs(List<Integer> res , int i , int n){
        if(i<=n)
           res.add(i);
        else //如果数字大于规定的数，返回上一层
        return ;
        for(int j=0;j<=9;j++){
            //进入下一层
            dfs(res,i*10+j,n);
        }
    }

    public List<Integer> lexicalOrder(int n){
        List<Integer> res = new ArrayList<>();
        //最开始的第一层
        for(int i=1;i<=9;i++){
            dfs(res, i, n);
        }
        return res;
    }
}
```



