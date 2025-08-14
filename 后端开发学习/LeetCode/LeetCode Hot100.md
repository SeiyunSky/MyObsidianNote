## 最大子数组和
![[Pasted image 20250626171621.png|400]]
```C++
我的思路：保存当前最大子数组大小
往前++，当当前和小于0，就丢弃当前记录的位置，换新位置
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int Max = num[0];
        int Temp = 0;
        for(int i = 0; i< nums.size();i++){
            Temp += nums[i];
            Max = max(Temp,Max);
            if(Temp < 0)
                Temp = 0;
            
        }
        return Max;
    } 
};
```

## 合并区间
![[Pasted image 20250626172756.png|500]]
```C++
我的想法：每次提取当前两个检查是否能合并，合并后再往后检查
直到不满足合并需求，放到新数组内
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        if (intervals.empty()) return {};

        // 按区间起始点排序
        sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
            return a[0] < b[0];
        });

        vector<vector<int>> ret;
        int i = 0;
        while (i < intervals.size()) {
            int first = intervals[i][0];
            int second = intervals[i][1];
            int j = i + 1;
            while (j < intervals.size() && second >= intervals[j][0]) {
                second = max(second, intervals[j][1]);
                j++;
            }
            ret.push_back({first, second});
            i = j;
        }
        return ret;
    }
};
```

```Java

class Solution {
    public int[][] merge(int[][] intervals) {
        if (intervals.length == 0) {
            return new int[0][0];
        }
    
        Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));
        List<int[]> merged = new ArrayList<>();
        
        int[] current = intervals[0];
        merged.add(current);
        for (int[] interval : intervals) {
            if (interval[0] <= current[1]) {
                current[1] = Math.max(current[1], interval[1]);
            } else {
                current = interval;
                merged.add(current);
            }
        }
        return merged.toArray(new int[merged.size()][]);
    }
}
```

## 轮转数组 
![[Pasted image 20250626180424.png|500]]
```C++
我的想法：
1. 直接脑部死亡一个一个换位置就完事了。 空间O（1）
2. 用另外一个数组去接数据。空间O（n）
3. 也是用另外一个，但辅助数组用K的大小接其中一半。空间O（k）
class Solution {
public:
    vector<int> rotate(vector<int>& nums, int k) {
        int n = nums.size();
        k %= n;  
        vector<int> asist(k);
        for (int i = n - k, j = 0; i < n; i++, j++) {
            asist[j] = nums[i];
        }
        for (int i = n - k - 1; i >= 0; i--) {
            nums[i + k] = nums[i];
        }
        
        for (int i = 0; i < k; i++) {
            nums[i] = asist[i];
        }
        return nums;
    }
};
```
## 除自身以外数组的乘积
![[Pasted image 20250626185626.png|500]]
```C++
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        vector<int> ret(nums.size(),0);
        long long temp = 1;
        for(int i = nums.size() - 1;i >= 0;i--){
            temp *= nums[i];
            ret[i] = temp;
        }
        temp =1;
        for(int i = 0;i < nums.size();i++){
            if(i+1 < nums.size()){
                 ret[i] = temp * ret[i + 1];
            }else{
                 ret[i] = temp;
            }
            temp *= nums[i];
        }
        return ret;
    }
};
```
## 缺失的第一个正数
![[Pasted image 20250630215323.png|500]]
```C++
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
         int n = nums.size();
         vector<int> hashNum(n+2,0);
         for(int i = 0;i < n;i++){
            if(nums[i] < n+1 && nums[i]>0){
                hashNum[nums[i]] = nums[i];
            }else if(nums[i]==1){
                hashNum[1] = 1;
            }
        }
         for(int i =1;i < n+2 ;i++){
            if(hashNum[i]!=i){
                return i;
            }
        }
         return n;
    }
};
```

## 矩阵置零
![[Pasted image 20250630223254.png|500]]
```C++
class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        int m = matrix.size();
        if (m == 0) return;
        int n = matrix[0].size();
        
        bool firstRowHasZero = false;
        bool firstColHasZero = false;
        
        for (int j = 0; j < n; j++) {
            if (matrix[0][j] == 0) {
                firstRowHasZero = true;
                break;
            }
        }
        
        for (int i = 0; i < m; i++) {
            if (matrix[i][0] == 0) {
                firstColHasZero = true;
                break;
            }
        }
        
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                if (matrix[i][j] == 0) {
                    matrix[i][0] = 0;  
                    matrix[0][j] = 0;  
                }
            }
        }
        
        for (int i = 1; i < m; i++) {
            if (matrix[i][0] == 0) {
                for (int j = 1; j < n; j++) {
                    matrix[i][j] = 0;
                }
            }
        }
        
        for (int j = 1; j < n; j++) {
            if (matrix[0][j] == 0) {
                for (int i = 1; i < m; i++) {
                    matrix[i][j] = 0;
                }
            }
        }
        
        if (firstRowHasZero) {
            for (int j = 0; j < n; j++) {
                matrix[0][j] = 0;
            }
        }

        if (firstColHasZero) {
            for (int i = 0; i < m; i++) {
                matrix[i][0] = 0;
            }
        }
    }
};
```


## 螺旋矩阵
![[Pasted image 20250701102118.png|500]]
```C++
我的想法：
    记录上下和左右，这两个参数，左到头就下，下到头就右，然后上下长度减少，右到头就上，同时左右减少两个数，然后上，然后上下减少，去左
    class Solution {

public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        int leftRight = matrix.size();
        int upDown = matrix[0].size();
        vector<int> ret;
        int i=0,j= 0;
    while(leftRight > 0 || upDown > 0){
    if(leftRight>0){
        for(;i < leftRight;i++){ //左
            ret.push_back(matrix[j][i]);
        }
        i--;
     }
     if(upDown>0){
        for(;j < upDown;j++){  //下
            ret.push_back(matrix[j][i]);
        }
        upDown--;j--;
      }
        for(int l =i;i > leftRight - l-1;i--){ //右
            ret.push_back(matrix[j][i]);
        }
        leftRight-=2;
        for(int l =j;j > upDown - l-1;j--){ //上
        ret.push_back(matrix[j][i]);
        }
        upDown--;
    }
    return ret;
       }
};

问题在我把updown一起更新了，就不太行
答案：
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        if (matrix.empty()) return {};
        
        int m = matrix.size(), n = matrix[0].size();
        int up = 0, down = m - 1, left = 0, right = n - 1;
        vector<int> ans;
        
        while (true) {
            for (int i = left; i <= right; i++) ans.push_back(matrix[up][i]);
            if (++up > down) break;
            for (int i = up; i <= down; i++) ans.push_back(matrix[i][right]);
            if (--right < left) break;
            for (int i = right; i >= left; i--) ans.push_back(matrix[down][i]);
            if (--down < up) break;
            for (int i = down; i >= up; i--) ans.push_back(matrix[i][left]);
            if (++left > right) break;
        }
        return ans;
    }
};
```

## 旋转图像
![[Pasted image 20250701111831.png|500]]
```C++
我的思路：四个角先换，1和2换，3和4换，然后1和3换
1 [l,l] 2 [l,p] 3 [p,p] 4 [p,l]
其中l每轮次+1，p每轮次-1
然后换中间的四个行，上和右换，下和左换，上下互换
因为矩阵是n*n,所以每次减小一圈，会导致模型收缩2的长度
对于特殊情况2*2，直接完成就行
对于其他情况，每次-2


class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int l = matrix.size();
        int n = l;
        int sizeMatrix = l;
        for()
    }
};
[1,2,3,4],
[5,6,7,8],
[9,10,11,12],
[13,14,15,16]

[13,5,9,1],
[14,10,6,2],
[15,11,7,3],
[16,8,12,4]

[13,9,5,1],
[14,10,6,2],
[15,11,7,3],
[16,12,8,4]
```

## 搜索二维矩阵 II
![[Pasted image 20250701144813.png||480]]
```C++
找到一个可以使得移动伴随的变化有区别的点
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        int m= matrix.size();
        int i = 0;
        int n =matrix[0].size()-1;
        while(i < m && n >= 0){
            if (matrix[i][n] == target) {
                return true;
            } else if (matrix[i][n] > target) {
                n--;
            } else {
                i++;
            }
        }
        return false;
    }
};```

## 相交链表
![[Pasted image 20250701151352.png]]
```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        int m = 0,n=0;
        ListNode *headM = headA;
        while(headM!=NULL){
            headM = headM->next;
            m++;
        }
        ListNode *headN = headB;
        while(headN!=NULL){
            headN = headN->next;
            n++;
        }
        int temp = 0;
        if(m>n){
            temp = m - n;
            headM = headA;
            headN = headB;
            while(temp--){
                headM = headM->next;
            }
            temp = n;
        }else if(n>m){
            temp = n - m;
            headN = headB;
            headM = headA;
            while(temp--){
                headN = headN->next;
            }
            temp = m;
        }
        if(n==m){
            temp = n;
            headM = headA;
            headN = headB;
        }
        while(temp--){
            if(headM == headN){
                cout<<headM->val<<endl;
                return headM;
            }else if(headM!=NULL&&headN!=NULL){
                headM = headM->next;
                headN = headN->next;
            }
        }
        return NULL;
    }
};
```

## 反转链表
![[Pasted image 20250701153532.png|500]]
```C++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */

class Solution {
public:
    ListNode* reverse(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        ListNode* newHead = reverse(head->next);
        head->next->next = head;
        head->next = nullptr;
        return newHead;
    }
    
    ListNode* reverseList(ListNode* head) {
        return reverse(head);
    }
};
```
## 删除链表的倒数第N个结点
![[Pasted image 20250704103125.png]]
```C++
class Solution {

public:

    ListNode* removeNthFromEnd(ListNode* head, int n) {

        ListNode* slow = head;

        ListNode* fast = head;

        if(head->next==nullptr||head==nullptr)

            return NULL;

  

        for(int i = 0;i < n;i++)

            fast = fast->next;

  

        if(fast==nullptr){

            return head->next;

        }

        while(fast->next!=nullptr)

        {

            slow = slow->next;

            fast = fast->next;

        }

        slow -> next = slow -> next -> next;

        return head;

    }

};
```

## 反转节点
```java
class Solution {
    public ListNode swapPairs(ListNode head) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode prev = dummy;
        
        while (head != null && head.next != null) {
            ListNode temp = head.next;
            head.next = temp.next;
            temp.next = head;
            prev.next = temp;
            prev = head;
            head = head.next;
        }   
        return dummy.next;
    }
}
```

## 字母异位词分组
![[Pasted image 20250704113011.png]]
```C++
代码解释：将字符串一个个取出，放在map中，键值是排序后的字符串，内容是原字符串
sort(key.begin(), key.end());
data[key].push_back(s);
这两行为原字符串排序并以排序字符串为key，将原字符串推入map中

//排序方法
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        unordered_map<string,vector<string>> data;
        for (const auto& s : strs) {
            auto key = s;
            sort(key.begin(), key.end());
            data[key].push_back(s);
        }
        vector<vector<string>> ret;
        for (const auto& p : data) {
            ret.push_back(p.second);
        }
        return ret;
    }
};

//​质数乘积法​
class Solution {
public:
    vector<vector<string>> groupAnagrams(vector<string>& strs) {
        int primes[26] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41,43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101};
        unordered_map<long, vector<string>> mp;   
        for (const string& s : strs) {
            long key = 1;
            for (char c : s) {
                key *= primes[c - 'a'];  
            }
            mp[key].push_back(s);
        }
        vector<vector<string>> ret;
        for (const auto& p : mp) {
            ret.push_back(p.second); 
        }
        return ret;
    }
};
```

## 合并K个升序链表
![[Pasted image 20250708185427.png|500]]
```C++
我的思路：按照list的长度，做一个败者树
然后互相比拼，非空最小的上去
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
    //PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
    //Java可以直接使用PriorityQueue，输入排序方法即可
    //Java利用poll和offer进行出入栈
        using Elem = pair<int ,ListNode>
        priority_queue<Elem,vector<Elem>,greater<Elem>> pq;
        for(ListNode* node :lists) if(node){
            pq.push({node->val,node});
        } 
        ListNode* head =nullptr;
        ListNode* ptr = nullptr;
        while(!pq.empty()){
            Elem elem = pq.top();
            pq.pop();
            if(!head){
                head = ptr = elem.second;
            }else{
                ptr ->next =elem.second;
                ptr =elem.second;
            }
            if(elem.second->next){
                pq.push({elem.second->next->val,elem.second->next});
            }
        }
            return head;
    }
};
```
## K个一组翻转链表
![[Pasted image 20250708202801.png|400]]
```C++
需要用五个指针，两个分别是头尾，两个负责前后跑，一个负责交换
class Solution {
public:
    ListNode* reverseKGroup(ListNode* head, int k) {
        int n=0;
        ListNode* cur=head;
        while(cur){
            n++;
            cur=cur->next;
        }
        ListNode* dummy=new ListNode(0);
        dummy->next=head;

        ListNode* pre=dummy;
        ListNode* p0=dummy;
        cur=head;
        for(int i=0;i<n/k;i++){
            int k0=k;
            while(k0--){
                ListNode* nxt=cur->next;
                cur->next=pre;
                pre=cur;
                cur=nxt;
            }
            ListNode* tmp=p0->next;
            tmp->next=cur;
            p0->next=pre;
            p0=tmp;
        }
        return dummy->next;
    }
};
```

## 随机链表的复制
![[Pasted image 20250709193645.png|500]]
```JAVA
class Solution {
    public Node copyRandomList(Node head) {
        if (head == null) return null;
        // 使用原节点作为key
        Map<Node, Node> map = new HashMap<>();
        Node dummy = new Node(0);
        Node cur = dummy;
        Node temp = head;
        // 第一遍：创建新节点并建立映射
        while (temp != null) {
            Node newNode = new Node(temp.val);
            map.put(temp, newNode);
            cur.next = newNode;
            cur = cur.next;
            temp = temp.next;
        }

        // 第二遍：处理random指针
        temp = head;
        cur = dummy.next;
        while (temp != null) {
            // 处理可能为null的random指针
            cur.random = temp.random != null ? map.get(temp.random) : null;
            temp = temp.next;
            cur = cur.next;
        }
        return dummy.next;
    }
}
```

## 最长连续序列
![[Pasted image 20250709203648.png]]
```JAVA
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> set = new HashSet<>();
        for(int num:nums){
            set.add(num);
        }
        int maxLength = 0;
        for (int num : set) {
            if (!set.contains(num - 1)) {
                int currentNum = num;
                int currentLength = 1;
                while (set.contains(currentNum + 1)) {
                    currentNum++;
                    currentLength++;
                }
                maxLength = Math.max(maxLength, currentLength);
            }
        }
        return maxLength;        
    }
}
```
## LRU缓存
![[Pasted image 20250710120403.png]]
A、方案：LinkedList解决
但这样不行，因为删除的复杂度是O(n)

需要自定义一个双向链表，利用存储key和node的map、
当map内存在时，读取node，把他放在头部
当map不存在时，读取链表最后一个元素，删除map内的他，修改头尾关系
所以难度就在于写一个双向链表的应用相关接口
```JAVA
class LRUCache {
//思路，头插，去尾，如果get到了，相当于进行一个头中置换
//如果没get到，相当于是新的东西，直接去尾部找，如果已经满了，那么就去尾
//那也就是说，我还需要存一个当前的capacity，每次插入更新此数值
    private final int capacity;
    private final LinkedList<Map.Entry<Integer, Integer>> list;
    public LRUCache(int capacity) {
        this.capacity =capacity;
        this.list = new LinkedList<>();
    }
    public int get(int key) {
        for(int i = 0; i < list.size(); i++) {
        Map.Entry<Integer, Integer> entry = list.get(i);
        if(entry.getKey() == key) {
            // 找到key，将其移到列表头部
            int value = entry.getValue();
            list.remove(i);
            list.addFirst(Map.entry(key, value));
            return value;
        }
        }
        // 没找到key
        return -1;
    }
    public void put(int key, int value) {
        for (int i = 0; i < list.size(); i++) {
        Map.Entry<Integer, Integer> entry = list.get(i);
        if (entry.getKey().equals(key)) {
            list.remove(i);
            list.addFirst(Map.entry(key, value));
            return;
        }
        }
        if (list.size() == capacity) {
        list.removeLast();
        }
        list.addFirst(Map.entry(key, value));
    }
    public int getCapaNow(){
        return this.capacity - list.size();
    }
}
```

## 排序链表
![[Pasted image 20250710213139.png|500]]
```JAVA
class Solution {
    public ListNode sortList(ListNode head) {
        //归并算法,拆开大于三个的东西，然后归并启动
        if(head==null || head.next == null)
        return head;
        ListNode slow = head,fast = head.next;
        while(fast!=null && fast.next!=null){
            slow = slow.next;
            fast = fast.next.next;
        }
        ListNode mid = slow.next;
        slow.next = null;
        ListNode l = sortList(head);
        ListNode r = sortList(mid);
        return merge(l,r);
    }
    
    public ListNode merge(ListNode l1,ListNode l2){
        ListNode dummy = new ListNode(0);
        ListNode curr = dummy;
        while (l1 != null && l2 != null) {
        if (l1.val < l2.val) {
            curr.next = l1;
            l1 = l1.next;
        } else {
            curr.next = l2;
            l2 = l2.next;
        }
        curr = curr.next;
        }
      curr.next = (l1 != null) ? l1 : l2;
      return dummy.next;
    }
}
```

## 快速排序
```JAVA
public class QuickSort {
    public static void quickSort(int[] arr) {
        if (arr == null || arr.length <= 1) {
            return; // 边界条件：空数组或单元素数组无需排序
        }
        quickSort(arr, 0, arr.length - 1); // 调用递归排序
    }
    
    private static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            // 分区操作，返回基准值的正确位置
            int pivotIndex = partition(arr, low, high);
            // 递归排序左子数组（小于基准值的部分）
            quickSort(arr, low, pivotIndex - 1);
            // 递归排序右子数组（大于基准值的部分）
            quickSort(arr, pivotIndex + 1, high);
        }
    }
    private static int partition(int[] arr, int low, int high) {
        int pivot = arr[high]; // 选择最右侧元素作为基准值
        int i = low - 1;       // i 是小于基准值的区域的边界
        for (int j = low; j < high; j++) {
            if (arr[j] < pivot) {
                i++;
                swap(arr, i, j); // 将小于基准值的元素交换到左侧
            }
        }
        swap(arr, i + 1, high);  // 将基准值放到正确位置
        return i + 1;            // 返回基准值的索引
    }
}
```
## 对称二叉树
![[Pasted image 20250715102815.png|400]]
```java
核心，return中递归+判断
class Solution {
    public boolean isSymmetric(TreeNode root) {
        return helper(root.left, root.right);
    }
    public boolean helper(TreeNode left, TreeNode right) {
        if (left==null &&right==null) return true;
        if (left==null || right==null) return false;
        return left.val == right.val && helper(left.left, right.right) && helper(left.right, right.left);
    }
}
```

## 二叉树的直径
![[Pasted image 20250715104441.png]]
```JAVA
class Solution {
    int ret = 0;
    public int diameterOfBinaryTree(TreeNode root) {
        //也就是递归寻找当前节点左右最深的情况，让每个节点都作为根节点去判定一次即可
        if(root!=null){
        ret = Math.max(ret,counter(root));
        ret = Math.max(ret,diameterOfBinaryTree(root.left));
        ret = Math.max(ret,diameterOfBinaryTree(root.right));}
        return ret;
    }
    private int counter(TreeNode root){
        if(root ==null)
            return 0;
        int l = check(root.left);
        int r = check(root.right);
        return l+r;
    }
    private int check(TreeNode node){
        if(node == null)
            return 0;
        return 1+Math.max(check(node.left),check(node.right));
    }
}
```
## 二叉树的层序遍历
![[Pasted image 20250722112314.png|500]]
```JAVA
使用双端链表，通过队列实现，进行层序遍历
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) {
            return result;
        }
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while (!queue.isEmpty()) {
            int levelSize = queue.size();
            List<Integer> currentLevel = new ArrayList<>();
            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.poll();
                currentLevel.add(node.val);
                if (node.left != null) {
                    queue.offer(node.left);
                }
                if (node.right != null) {
                    queue.offer(node.right);
                }
            }
            result.add(currentLevel);
        }
        return result;
    }
}
```

## 二叉树展开为链表
![[Pasted image 20250722113837.png|400]]
```JAVA
利用前驱节点特性，进行原地转换
找到当前节点子树最右节点，即是当前节点右树的父节点
class Solution {
    public void flatten(TreeNode root) {
        TreeNode curr = root;
        while (curr != null) {
            // 如果有左子树
            if (curr.left != null) {
                TreeNode predecessor = curr.left;
                while (predecessor.right != null) {
                    predecessor = predecessor.right;
                }
                predecessor.right = curr.right;
                curr.right = curr.left;
                curr.left = null;
            }
            curr = curr.right;
        }
    }
}
```

## 二叉树的右视图
![[Pasted image 20250722115814.png|400]]
```JAVA
 //相当于我要找到每一层最左边的东西
 //也就是，层序遍历的同时
 //当i到了levelsize-1的时候，记录当前数值
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> ret = new ArrayList();
        if(root == null)
            return ret;
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        while(!queue.isEmpty()){
            int levelSize = queue.size();
            for(int i = 0 ; i<levelSize;i++){
                TreeNode node = queue.poll();
                if (i == levelSize - 1){
                    ret.add(node.val);
                }

                if (node.left != null) {
                    queue.offer(node.left);
                }
                if (node.right != null) {
                    queue.offer(node.right);
                }
            }
        }
        return ret;        
    }
}
```

## 

```JAVA
class Solution {
    // 使用一个变量来记录中序遍历过程中，前一个节点的值。
    // 使用 Long 类型是为了处理边界情况，比如当节点的 val 是 Integer.MIN_VALUE 时。
    // 初始化为一个比所有 int 都小的值。
    private long preValue = Long.MIN_VALUE;
    public boolean isValidBST(TreeNode root) {
        if (root == null) {
            return true;
        }
        if (!isValidBST(root.left)) {
            return false;
        }
        if (root.val <= preValue) {
            return false;
        }
        preValue = root.val;
        return isValidBST(root.right);
    }
}
```

## 二叉树的最近公共祖先
![[Pasted image 20250801104256.png|400]]
```JAVA
通过左右中的后序遍历提供回溯数据
class Solution {
    TreeNode travalsal(TreeNode node,TreeNode q,TreeNode p){
        if(node==null)
            return null;
        if(node==p||node==q)
            return node;
        TreeNode left =travalsal(node.left,q,p);
        TreeNode right =travalsal(node.right,q,p);
        if(left!=null && right!=null)
            return node;
        if(left==null && right!=null)
            return right;
        else if(left!=null && right==null)
            return left;
        return null;
    }
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        return travalsal(root,p,q);
    }
}
```

## 路径总和III 
![[Pasted image 20250801111552.png|400]]
**双重递归**
```Java
class Solution {
    public int pathSum(TreeNode root, int targetSum) {
        if (root == null) return 0;
        // 从根节点开始向下找或者从其他非根节点开始向下找
        int result = pathSumStartWithRoot(root, targetSum) + pathSum(root.left, targetSum) + pathSum(root.right, targetSum);
        return result;
    }
    private int pathSumStartWithRoot(TreeNode root, int targetSum) {
        if (root == null) return 0;
        // 以当前节点作为起始点让路径和先归0，重新开始向下找路径和是否与targetSum匹配
        int result = 0;
        if (root.val == targetSum) result++;
        // 探索当前节点向下所有左右子树，targetSum与对应值做差，递归到二者相等记录找到一条路径，探底后马上收敛
        result += pathSumStartWithRoot(root.left, targetSum - root.val) + pathSumStartWithRoot(root.right, targetSum - root.val);
        return result;
    }
}
```
**哈希表前缀和方法**
```java
class Solution {
    public int pathSum(TreeNode root, int targetSum) {
        // 哈希表记录前缀和及其出现次数
        Map<Long, Integer> prefixSumCount = new HashMap<>();
        prefixSumCount.put(0L, 1); // 初始化：前缀和为0的路径出现1次（空路径）
        return dfs(root, 0, targetSum, prefixSumCount);
    }
    private int dfs(TreeNode node, long currentSum, int targetSum, Map<Long, Integer> prefixSumCount) {
        if (node == null) return 0;
        currentSum += node.val; // 更新当前路径和
        // 检查是否存在满足条件的前缀和
        int res = prefixSumCount.getOrDefault(currentSum - targetSum, 0);
        // 将当前前缀和存入哈希表
        prefixSumCount.put(currentSum, prefixSumCount.getOrDefault(currentSum, 0) + 1);
        // 递归处理左右子树
        res += dfs(node.left, currentSum, targetSum, prefixSumCount);
        res += dfs(node.right, currentSum, targetSum, prefixSumCount);
        // 回溯：移除当前前缀和，避免影响其他分支
        prefixSumCount.put(currentSum, prefixSumCount.get(currentSum) - 1);
        return res;
    }
}
```

## 二叉树中的最大路径和
![[Pasted image 20250801115204.png|400]]
**思路是找寻当前根节点向下的子树最大贡献值**：
```java
class Solution {
    int maxNum = Integer.MIN_VALUE;
    public int maxPathSum(TreeNode root) {
        search4Best(root);
        return maxNum;
    }
    public int search4Best(TreeNode root){
        if(root == null)
            return 0;
        int leftGain = Math.max(search4Best(root.left),0);
        int rightGain = Math.max(search4Best(root.right),0);
        int res = root.val+leftGain +rightGain;
        maxNum = Math.max(maxNum,res);
        return root.val + Math.max(leftGain, rightGain);
    }
}
```