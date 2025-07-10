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

## **轮转数组** 
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

## **Java Set 接口及实现类语法速查**
**核心实现类**  
- `HashSet`：哈希表实现，无序，O(1)操作  
- `LinkedHashSet`：保留插入顺序，哈希表+链表  
- `TreeSet`：红黑树实现，自动排序，O(log n)操作  
**基础操作**  
```java
Set<String> set = new HashSet<>();
set.add("A");                 // 添加元素
set.remove("A");              // 删除元素
set.contains("A");            // 判断存在
set.size();                   // 获取大小

// for-each
for (String s : set) { ... }
// Iterator
Iterator<String> it = set.iterator();
set.forEach(s -> System.out.println(s));

set1.addAll(set2);      // 并集
set1.retainAll(set2);   // 交集
set1.removeAll(set2);   // 差集

// Treeset中自然排序
Set<Integer> treeSet = new TreeSet<>(); 
// 逆序排序
Set<String> customSet = new TreeSet<>(Comparator.reverseOrder());
//自定义排序，用其中的对象属性排序
Set<String> set = new TreeSet<>(
    (s1, s2) -> s1.length() - s2.length()
);
```
## **Java Map核心实现类速查**  
- **HashMap**：哈希表实现，无序，O(1)操作  
- **LinkedHashMap**：保留插入/访问顺序，哈希表+链表  
- **TreeMap**：红黑树实现，按键自动排序，O(log n)操作  
- **ConcurrentHashMap**：线程安全分段哈希表  

**基础操作**  
```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);          // 插入
map.get("A");             // 查询
map.remove("A");          // 删除
map.forEach((k,v) -> ...); // 遍历

// 自然排序
Map<String, Integer> treeMap = new TreeMap<>(); 
// 自定义排序
Map<String, Integer> customMap = new TreeMap<>(
    Comparator.comparingInt(String::length)
);

// 按值排序（需转List）
map.entrySet().stream()
   .sorted(Map.Entry.comparingByValue())
   .collect(Collectors.toList());


```

## LRU缓存
![[Pasted image 20250710120403.png]]
```JAVA
class LRUCache {
    public LRUCache(int capacity) {

    }
    public int get(int key) {

    }
    public void put(int key, int value) {

    }
}
```