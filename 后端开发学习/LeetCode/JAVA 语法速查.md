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
map.containsKey(key)/containsValue(value) //检查存在性
int size = map.size();
boolean isEmpty = map.isEmpty();
map.replace("B", 3); //替换已存在的键

// 自然排序
Map<String, Integer> treeMap = new TreeMap<>(); 
// 自定义排序
Map<String, Integer> customMap = new TreeMap<>(
    Comparator.comparingInt(String::length)
);

直接操作某个键值对，用的是地址
Map.Entry<Integer, Integer> entry = list.get(i);

// 按值排序（需转List）
map.entrySet().stream()
   .sorted(Map.Entry.comparingByValue())
   .collect(Collectors.toList());

//获得所有的key
map.keySet();
```

## **Java List 接口及实现类语法速查​**​
​**​核心实现类​**​
- `ArrayList`：动态数组实现，随机访问快（O(1)），增删慢（O(n)）
- `LinkedList`：双向链表实现，增删快（O(1)），随机访问慢（O(n)）

```JAVA
List<String> list1 = new ArrayList<>();  // 空列表

List<Integer> list2 = new LinkedList<>(Arrays.asList(1, 2, 3));  // 初始化元素

LinkedList<String> linkedList = new LinkedList<>();
这些FL方法，必须直接以Linklist来构造，不然没有，还是得以list来做
linkedList.addFirst("A");    // 头部插入 O(1)
linkedList.addLast("B");     // 尾部插入 O(1)
linkedList.removeFirst();    // 头部删除 O(1)

List<String> list = new ArrayList<>();

// 增
list.add("Java");            // 末尾添加
list.add(0, "Python");       // 指定位置插入
list.addAll(Arrays.asList("C++", "Go"));  // 添加集合

// 删
list.remove(0);              // 按索引删除
list.remove("Java");         // 按元素删除
list.removeAll(List.of("C++", "Go"));  // 删除指定集合元素
list.clear();                // 清空列表

// 改
list.set(1, "Rust");         // 修改指定位置元素

// 查
String s = list.get(0);      // 获取索引元素
boolean hasJava = list.contains("Java");  // 判断存在
int index = list.indexOf("Python");  // 查找索引
int size = list.size();      // 获取大小
boolean isEmpty = list.isEmpty();  // 判空

// for 循环
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}
// 增强 for 循环
for (String item : list) {
    System.out.println(item);
}

// Iterator
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}

// 自然排序（元素需实现 Comparable）
Collections.sort(list);  // 升序
// 自定义排序（Comparator）
list.sort((s1, s2) -> s1.length() - s2.length());  // 按字符串长度排序
list.sort(Comparator.reverseOrder());  // 逆序排序
// 对象属性排序
List<Person> people = new ArrayList<>();
people.sort(Comparator.comparing(Person::getName));  // 按 name 排序

List<String> subList = list.subList(1, 3);  // 截取子列表（左闭右开）

// List → 数组
String[] array = list.toArray(new String[0]);
// 数组 → List
List<String> immutableList = Arrays.asList("A", "B", "C");
// 深拷贝
List<String> copy = new ArrayList<>(list);
```

## 双向链表
```JAVA
class Node<T> {
    T data;          // 存储数据
    Node<T> prev;    // 前驱指针
    Node<T> next;    // 后继指针

    public Node(T data) {
        this.data = data;
        this.prev = null;
        this.next = null;
    }
}
```

```java
双向链表类
public class DoublyLinkedList<T> {
    private Node<T> head;  // 头节点
    private Node<T> tail;  // 尾节点
    private int size;      // 链表长度

    public DoublyLinkedList() {
        this.head = null;
        this.tail = null;
        this.size = 0;
    }

    // 判断链表是否为空
    public boolean isEmpty() {
        return size == 0;
    }

    // 返回链表长度
    public int size() {
        return size;
    }

    // 在链表头部插入节点
    public void addFirst(T data) {
        Node<T> newNode = new Node<>(data);
        if (isEmpty()) {
            head = tail = newNode;
        } else {
            newNode.next = head;
            head.prev = newNode;
            head = newNode;
        }
        size++;
    }

    // 在链表尾部插入节点
    public void addLast(T data) {
        Node<T> newNode = new Node<>(data);
        if (isEmpty()) {
            head = tail = newNode;
        } else {
            tail.next = newNode;
            newNode.prev = tail;
            tail = newNode;
        }
        size++;
    }

    // 删除链表头部节点
    public T removeFirst() {
        if (isEmpty()) {
            throw new IllegalStateException("链表为空");
        }
        T data = head.data;
        head = head.next;
        if (head != null) {
            head.prev = null;
        } else {
            tail = null; // 链表已空
        }
        size--;
        return data;
    }

    // 删除链表尾部节点
    public T removeLast() {
        if (isEmpty()) {
            throw new IllegalStateException("链表为空");
        }
        T data = tail.data;
        tail = tail.prev;
        if (tail != null) {
            tail.next = null;
        } else {
            head = null; // 链表已空
        }
        size--;
        return data;
    }

    // 删除指定值的节点（从头开始搜索）
    public boolean remove(T data) {
        Node<T> current = head;
        while (current != null) {
            if (current.data.equals(data)) {
                if (current == head) {
                    removeFirst();
                } else if (current == tail) {
                    removeLast();
                } else {
                    current.prev.next = current.next;
                    current.next.prev = current.prev;
                    size--;
                }
                return true;
            }
            current = current.next;
        }
        return false;
    }

    // 查找节点是否存在
    public boolean contains(T data) {
        Node<T> current = head;
        while (current != null) {
            if (current.data.equals(data)) {
                return true;
            }
            current = current.next;
        }
        return false;
    }

    // 打印链表（从头到尾）
    public void printForward() {
        Node<T> current = head;
        while (current != null) {
            System.out.print(current.data + " -> ");
            current = current.next;
        }
        System.out.println("null");
    }

    // 打印链表（从尾到头）
    public void printBackward() {
        Node<T> current = tail;
        while (current != null) {
            System.out.print(current.data + " -> ");
            current = current.prev;
        }
        System.out.println("null");
    }
}
```

## **Java Queue 接口及实现类语法速查**
```JAVA

class QueueCheatsheet {

    /**
     * ================================
     * ||       核心实现类 (Core)       ||
     * ================================
     *
     * - LinkedList:   基于双向链表，也实现了 Deque 接口，在两端增删效率高 (O(1))。
     * - ArrayDeque:   基于可动态调整大小的数组，也实现了 Deque 接口，在两端增删的平摊时间复杂度为 O(1)。通常比 LinkedList 更高效。
     * - PriorityQueue: 基于优先堆的无界优先级队列。元素按自然顺序或自定义 Comparator 排序。
     */
    void coreImplementations() {
        // 作为队列使用时，推荐使用 ArrayDeque
        Queue<String> queue1 = new ArrayDeque<>();  // 空队列

        // LinkedList 也可以作为队列
        Queue<Integer> queue2 = new LinkedList<>(Arrays.asList(1, 2, 3));  // 初始化元素

        // 优先级队列，默认是小顶堆（自然排序）
        Queue<Integer> priorityQueue = new PriorityQueue<>();

        // ArrayDeque 作为双端队列，功能更强大
        Deque<String> deque = new ArrayDeque<>();
        deque.addFirst("A"); // 头部插入
        deque.addLast("B");  // 尾部插入
    }


    public static void main(String[] args) {
        Queue<String> queue = new ArrayDeque<>();

        // ================================
        // ||         增 (Offer/Add)       ||
        // ================================
        // offer(E e): 将元素添加到队尾，如果队列容量有限且已满，返回 false。推荐使用。
        queue.offer("Java");
        queue.offer("Python");
        // add(E e):    将元素添加到队尾，如果队列容量有限且已满，会抛出 IllegalStateException 异常。
        queue.add("Go");


        // ================================
        // ||         删 (Poll/Remove)     ||
        // ================================
        // poll():   获取并移除队首元素，如果队列为空，返回 null。推荐使用。
        String polledElement = queue.poll(); // "Java"
        System.out.println("Poll 之后: " + queue + ", 被移除的元素: " + polledElement); // [Python, Go]
        // remove(): 获取并移除队首元素，如果队列为空，会抛出 NoSuchElementException 异常。
        String removedElement = queue.remove(); // "Python"


        // ================================
        // ||         查 (Peek/Element)    ||
        // ================================
        queue.offer("Rust");
        System.out.println("再次添加元素后: " + queue); // [Go, Rust]
        // peek():    获取但不移除队首元素，如果队列为空，返回 null。推荐使用。
        String peekedElement = queue.peek(); // "Go"
        System.out.println("Peek 之后: " + queue + ", 查看的元素: " + peekedElement); // [Go, Rust]
        // element(): 获取但不移除队首元素，如果队列为空，会抛出 NoSuchElementException 异常。
        String element = queue.element(); // "Go"
        System.out.println("Element 之后: " + queue + ", 查看的元素: " + element); // [Go, Rust]

        // 其他常用方法
        int size = queue.size();            // 获取大小
        boolean isEmpty = queue.isEmpty();  // 判空
        queue.clear();                      // 清空队列


        // ================================
        // ||         遍历 (Iteration)     ||
        // ================================
        Queue<String> travelQueue = new ArrayDeque<>(Arrays.asList("A", "B", "C"));
        // 增强 for 循环
        System.out.print("增强 for 循环遍历: ");
        for (String item : travelQueue) {
            System.out.print(item + " "); // A B C
        }
        System.out.println();

        // Iterator (注意：遍历不保证顺序，特别是对于 PriorityQueue)
        System.out.print("Iterator 遍历: ");
        Iterator<String> it = travelQueue.iterator();
        while (it.hasNext()) {
            System.out.print(it.next() + " "); // A B C
        }
        System.out.println();


        // ================================
        // ||   排序 (PriorityQueue)      ||
        // ================================
        // 自然排序（元素需实现 Comparable），默认小顶堆
        PriorityQueue<Integer> pqNatural = new PriorityQueue<>();
        pqNatural.offer(5);
        pqNatural.offer(2);
        pqNatural.offer(8);
        System.out.println("PriorityQueue 自然排序 (poll): " + pqNatural.poll() + ", " + pqNatural.poll() + ", " + pqNatural.poll()); // 2, 5, 8

        // 自定义排序（Comparator），通过构造函数传入
        // 按字符串长度排序
        PriorityQueue<String> pqCustom = new PriorityQueue<>((s1, s2) -> s1.length() - s2.length());
        pqCustom.offer("JavaScript");
        pqCustom.offer("Go");
        pqCustom.offer("Java");
        System.out.print("PriorityQueue 按长度排序 (poll): ");
        while(!pqCustom.isEmpty()) {
            System.out.print(pqCustom.poll() + " "); // Go Java JavaScript
        }
        System.out.println();

        // 大顶堆 (逆序排序)
        PriorityQueue<Integer> pqReverse = new PriorityQueue<>(Comparator.reverseOrder());
        pqReverse.offer(5);
        pqReverse.offer(2);
        pqReverse.offer(8);
        System.out.println("PriorityQueue 逆序排序 (poll): " + pqReverse.poll() + ", " + pqReverse.poll() + ", " + pqReverse.poll()); // 8, 5, 2

        // 对象属性排序
        PriorityQueue<Person> people = new PriorityQueue<>(Comparator.comparing(Person::getName));
        people.offer(new Person("Bob"));
        people.offer(new Person("Alice"));
        people.offer(new Person("Charlie"));
        System.out.print("PriorityQueue 按对象属性排序 (poll): ");
        while(!people.isEmpty()){
            System.out.print(people.poll().getName() + " "); // Alice Bob Charlie
        }
        System.out.println();
    }
}
```