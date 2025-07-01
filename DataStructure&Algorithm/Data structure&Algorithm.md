# 数组 与 链表



## 数组



优点

- 支持随机访问：O(1)时间内访问任何元素
- 缓存局部性：当访问数组元素时，计算机不仅会加载它，还会缓存其周围的数据，因为数组是连续空间存储

缺点

- 插入与删除效率低O(n)
- 长度固定
- 空间浪费



C++ 动态数组 vector



## 链表



链表结构体

```C++
struct ListNode {
    int val; //节点值
    ListNode* next; //指向下一节点的指针
    ListNode(int x): val(x), next(nullptr) {}; //构造函数
}
```



### 常见链表类型



- 单向链表
- 双向链表
- 环形链表







# 栈 与 队列



## 栈



## 队列



- 基于环形数组实现的队列



### 双向队列



# 哈希表

通过建立键key与值value之间的映射，来实现高效的元素查询。





## 哈希冲突



哈希表就是将一个较大的输入空间映射到一个较小的输出空间中，这样就可能会出现多个输入对应一个输出的情况，这就是哈希冲突。



```C++
/* 哈希函数 */
int hashFunc(int key) {
	int index = key % 100; //索引值为[0, 99]
	return index;
}
/*
12836 % 100 = 36
20336 % 100 = 36
*/
```



### ？如何解决哈希冲突

- 扩容 -> 减少哈希冲突

  - 优：简单粗暴
  - 缺：效率低

- 链式地址：将单个元素转换为链表，键值对作为链表节点，发生哈希冲突的键值对存储在同一链表中

  - 缺：占用空间增大；查询效率降低

- 开放寻址（都不能直接删除元素）

  1. 线性探测
  
     - 通过哈希函数计算桶索引，若桶内已有元素，则产生哈希冲突。
  
     - 从下一个桶开始往后线性遍历，直到找到空桶，将元素插入其中。
  

     - 新的问题：

       - 聚集现象：连续位置发生哈希冲突的可能性增大

       - 也不能直接删除元素，不然留下的空桶可能就导致程序误判：产生哈希冲突了的元素不存在

  
     - 引入==懒删除机制==
  
       - 利用一个 *TOMBSTONE* 来标记这个桶。其中 *None* 和 *TOMBSTONE* 都代表空桶。
  
       - 线性探测，在探测到 *TOMBSTONE* 后继续向后遍历
  
       - 如果要删除某个桶的元素，我们就将这个桶的指针给标记成TOMESTONE，表示空桶。
  
  
     - 这样做的后果是：可能要跳过多个 *TOMBSTONE* 来能找到目标元素
  
  2. 开放寻址：平方探测
  
     - 跳过“探测次数平方”的步数
  
     - 优：跳过更大的距离使数据分布更加均匀，有助于缓解聚集现象
  
     - 缺：由于平方的增长，即使有空位平方探测也可能访问不到
  
  3. 多次哈希
  
     - 使用多个哈希函数进行探测
  
     - 缺：增加计算量





# 树



## 二叉树



二叉树节点结构体

```c++
struct TreeNode {
    int val; //节点值
    TreeNode* left; //左子节点指针
    TreeNode* right; //右子节点指针
    TreeNode(int x): val(x), left(nullptr), right(nullptr) {};
}
```



### 二叉树常见术语



- 节点的度：节点的子节点的数量，度的取值范围为0、1、2。
- 二叉树的高度：从根节点到最远子节点的==边的数量==
- 节点的深度：从根节点到该节点的==边的数量==
- 节点的高度：从距离该节点最远的叶子节点到该节点的边的数量





### 常见二叉树



- 完美二叉树/满二叉树：所有层的节点都是满的
- 完全二叉树：最底层的节点依次靠左填充
- 完满二叉树：除了叶子节点，任意节点都有两个子节点
- 平衡二叉树：任意节点的左子树高度与右子树高度的差值不超过1
- 二叉搜索树：左子树所有节点的值 < 根节点的值 < 右子树所有节点的值



### 二叉树遍历



- 层序遍历/广度优先搜索(Breadth-first search, BFS)

```C++
/* 层序遍历 */
vector<int> levelOrder(TreeNode* root) {
    queue<TreeNode*> queue; 
    queue.push(root); //初始化队列
    vector<int> vec; //初始化列表，保存遍历序列
    while (!queue.empty()) {
        TreeNode* node = queue.front();
        queue.pop();
        vec.push_back(node->val);
        if (node->left)
            queue.push(node->left);
       	if (node->right)
            queue.push(node->right);
    }  
}
/* 时间复杂度:O(n) */
/* 空间复杂度:O(n) */
```

- 时间复杂度：$O(n)$
- 空间复杂度：$O(n)$





- 深度优先遍历(depth-first search, DFS)/前序/中序/后序

```c++
/* 前序遍历 */
void preOrder(TreeNode* root) {
    if (!root)
        return;
    vec.push_back(root->val);
    preOrder(root->left);
    preOrder(root->right); //根左右
}
```

- 时间复杂度：$O(n)$
- 空间复杂度：$O(n)$



### 二叉搜索树



- 左子树中所有节点的值 < 根节点的值 < 右子树所有节点的值
- 任意左、右子树也是二叉搜索树



#### 查找节点

```c++
TreeNode* search(int num) {
    TreeNode* cur = root;
    while (!cur) {
        if (cur->val < num)
            cur = cur->right;
        else if (cur->val > num)
            cur = cur->left;
        else
            break;
    }
    return cur;
}
```



#### 插入节点





#### 删除节点

二叉搜索树的中序遍历是升序的。





















# 堆

满足特定条件的==完全二叉树==



- 小顶堆：任意节点的值 $\leq$ 其子节点的值
- 大顶堆：任意节点的值 $\geq$ 其子节点的值



C++ 优先队列（priority queue，默认是最大的在前面，所以是大顶堆）



假设一棵==完全二叉树==有 $n$ 个节点，高度为 $h$，那一定满足不等式 $2^{h}\leq n < 2^{h+1}$ ，因此，树的高度为 $O(\log n)$





元素入堆（$O(\log n)$）

1. 添加元素到堆底（因为是采用数组存储，所以时间复杂度是 $O(1)$）
2. 从底到顶执行堆化（时间复杂度是 $O(\log n)$）

堆顶元素出堆（$O(\log n)$）

1. 交换堆顶元素与堆底元素
2. 弹出堆底元素
3. 从顶至底执行堆化

建堆

- 遍历列表，依次执行入堆操作（n个元素做入堆操作）
- 时间复杂度：O($n\log n$)

建堆也可以进一步优化成 $O(n)$



# 图



链表/数组：线性关系

树：分治关系



图的分类

- 按边有向和无向
  - 有向图
  - 无向图
- 按所有顶点是否连通
  - 连通图
  - 非连通图
- 为边添加权重
  - 有权图



图的相关术语

- 度：一个顶点拥有的边数
- 入度：有多少边指向该顶点



图的邻接矩阵存储

```C++
/* 基于邻接矩阵的图存储 */
typedef char VertexType; // 顶点类型
typedef int EdgeType; // 边上权值
#define MAXVEX 100 // 最大顶点数
#define INFINITY 65535 //表示无穷大
typedef struct {
	VertexType vexs[MAXVEX]; /* 顶点表 */
	EdgeType arc[MAXVEX][MAXVEX]; /* 邻接矩阵，可看做边表 */
	int numNodes, numEdges; /* 图中的顶点数 和 边数 */
} MGraph;
```



创建一张图所需时间复杂度 $O(n^{2})$



图的邻接表存储

```C++
/* 基于邻接表的图存储 */
typedef struct EdgeNode { // 边表结点
	int adjvex;
	EdgeType info; // 用于存储权值
	struct EdgeNode* next; // 指向下一个节点
} EdgeNode;

typedef struct VertexNode { // 顶点表节点
	VertexType data; // 存储顶点信息
	EdgeNode *firstedge; // 边表头指针
} VertexNode, AdjList[MAXVEX];

typedef struct {
	AdjList agjList;
	int numNodes, numEdges;
} GraphAdjList;
```









图的遍历

- 广度优先遍历（BFS）

算法实现

```C++
/* 广度优先遍历 */
//使用邻接表
```



- 深度优先遍历







# 20250611

## **排序**

- 稳定性：排序后相等元素的相对位置不发生变化。



### **选择排序**

基本操作：

1. 通过n-1次比较，从n个记录中选出最小的记录，将它与第1个交换
2. 再通过n-2次比较，从剩余的n-1个记录中找出次小的记录，将它与第2个交换
3. 重复上述操作，共需n-1次

#### 思考

n个元素

第1次 确定第1个位置（最小的） 比较n-1次

第2次 确定次小的 剩下n-1个元素 比较n-2次

第n-1次 剩下2个元素 比较1次 排序结束

```c++
//假设数组长度为n,对数组从小到大进行选择排序
void selectionSort(vector<int>& nums)
    int n = nums.size();
    for (int i = 0; i < n-1; ++i) { //外循环n-1次，因为只要确定前n-1个元素的位置
        int minIndex = i;
        for (int j = i+1; j < n; ++j) { 
            if (nums[j] < nums[i]) {
               minIndex = j;
            }
        }
        if (minIndex != i) //找到了更小的值
        	swap(num[i], num[minIndex]);
    }
}
```

- 时间复杂度：$O(n^{2})$

- 空间复杂度：$O(1)$
- 非稳定排序



### **冒泡排序**

基本思想：两两比较，如果发生逆序则交换，直到所有记录都排好序为止


#### 思考

n个元素

第1趟 比较n-1次 确定最后一个大的位置

第2趟 比较n-2次 确定次大的位置

第n-1趟 比较1次 确定第1个位置

```C++
void bubbleSort(vector<int>& vec)
{
	int n = vec.size();
	for (int i = 0; i < n - 1; ++i) //一共n-1趟
	{
        bool swapped = false; //提高效率
		for (int j = 0; j < n - i - 1; ++j)
		{
			if (vec[j] > vec[j+1]) //发生逆序
			{
				swap(vec[j], vec[j+1]);
                swapped = true; //表示发生了交换
			}
		}
        if (!swapped) //如果某一轮没有发生交换，说明数组已经有序
            break;
	}
}
```

优点：每趟结束，不仅能挤出一个最大值到后面，同时还能部分理顺其他元素

如何提高效率？

一旦某一趟比较时不出现交换记录，说明已经排好序了，可以立即结束排序。

- 时间复杂度：$O(n^{2})$

- 空间复杂度：$O(1)$
- 稳定排序



### **插入排序**

初始阶段第一张牌已经排好序，所以外部循环是遍历剩下的n-1张牌。

抽出要排序的一张牌（暂存）

```c++
void insertionSort(vector<int>& nums) {
    int n = nums.size();
    for (int i = 1; i < n; ++i) { //初始时刻假设第一张牌已经排好序
        int base = nums[i], j = i-1; //内循环：将base插入到已排序区间[0, i-1]的正确位置
        while (j >= 0 && nums[j] > base) {
            nums[j+1] = nums[j]; //将元素右移一位
            --j;
        }
        nums[j+1] = base;    
    }
}
```

- 时间复杂度：$O(n^{2})$<!-- 内层循环最差情况下完全逆序，1+2...+(n-1) -->
- 空间复杂度：$O(1)$
- 稳定排序



==插入排序==基于赋值操作，因此比==冒泡排序==基于临时变量的交换操作开销要更小。

### **快排**

基于分治策略。

```c++
/* 哨兵划分 */
int partition(vector<int>& nums, int left, int right) {
    int i = left, j =right; //nums[left]为基准数
    while (i < j) {
        while (i < j && nums[j] >= nums[left])
            --j; //从右向左找首个小于基准数的元素
        while (i < j && nums[i] <= nums[left])
            ++i; //从左向右找首个大于基准数的元素
        swap(nums[i], nums[j]);
    }
    swap(nums[i], nums[left]);
    return i; //返回基准值索引
}
```

哨兵划分结束后，原数组被划分成三部分：左子数组、基准数、右子数组，左子数组元素 $\leq$ 基准数 $\leq$ 右子树组。

```c++
void quickSort(vector<int>& nums, int left, int right) {
    if (left == right)
        return; //子数组长度为1时便停止递归
    int piovt = partition(nums, left, right); //piovt中心点
    quickSort(nums, left, piovt-1); //递归左子数组
    quickSort(nums, piovt+1, right);
}
```

- 时间复杂度：$O(n\log n)$ <!-- 递归深度为$\log n$，每层最多循环n -->
- 空间复杂度：$O(n)$ <!-- 栈帧空间 -->
- 非稳定排序

当输入数组倒序时，快排就退化成==冒泡排序==。

为了提高效率，选取三个候选元素（数组的首、尾、中点元素）的中位数作为==基准数==。



### **归并排序**

1. 划分阶段：通过递归不断将数组从中点处划开，长数组排序问题转换成短数组排序问题。
2. 合并阶段：子数组长度为1时终止划分，反过来开始有序合并。



```c++
/* 合并左子数组和右子树组 */
void merge(vector<int>& nums, int left, int mid, int right) {
    //左子数组区间为[left, mid]， 右子树组区间为[mid+1, right]
    vector<int> tmp(right-left+1); //临时数组暂存合并后的结果
    int i = left, j = mid+1, k = 0;
    //当左右数组都还有元素，进行比较，并将较小的元素复制到临时数组中
    while (i <= mid && j <= right) { 
        if (nums[i] < nums[j]) {
            tmp[k] = nums[i];
            ++i;
        }
        else {
            tmp[k] = nums[j];
            ++j;
        }
        ++k;
    } //执行完后要么左数组和右数组的值都已经被复制了，要么可能还剩下元素
    while (i <= mid) {
        tmp[k] = nums[i];
        ++i, ++k;
    }
    while (j <= right) {
        tmp[k] = nums[j];
        ++j, ++k;
    }
    for (k = 0; k < tmp.size(); ++k) {
        nums[left+k] = tmp[k]; //复制回原数组
    }
}

/* 归并排序 */
void mergeSort(vector<int>& nums, int left, int right) {
    if (left == right)
        return; //子数组长度为1时终止递归
    int mid = left + (right - left)/2; //计算中点
    mergeSort(nums, left, mid); //递归左子数组
    mergeSort(nums, mid+1, right);
    merge(nums, left, mid, right); //合并
}
```

- 时间复杂度：$O(n\log n)$

- 空间复杂度：$O(n)$

- 稳定排序



适合==链表排序==。



### **堆排序** 

排序后的数组从小到大。

1. 建堆、执行堆化
2. 弹出元素，堆化

```c++
/* 大顶堆由顶至底执行堆化 */
void siftDown(vector<int>& nums, int n, int i) { //堆的长度为n，从节点i开始，从顶至底堆化
    while (true) {
        int l = 2*i + 1; //左孩子下标
        int r = 2*i + 2; //右孩子下标
        int maxIndex = i; //maxIndex表示子树中最大的值
        if (l < n && nums[l] > nums[maxIndex])
            maxIndex = l;
        if (r < n && nums[r] > nums[maxIndex])
            maxIndex = r;
        if (maxIndex == i) { //无需堆化
            break;
        }
        swap(nums[maxIndex], nums[i]); //保证子树的父节点>叶子结点
        i = maxIndex; //继续向下执行堆化
    }
}

/* 堆排序 */
void heapSort(vector<int>& nums) {
    //堆化：除叶子节点之外的所有节点
    for (int i = nums.size()/2-1; i >= 0; --i) { 
        //假设根节点下标为i, 左孩子下标则为2i+1， 右孩子下标为2i+2，
        //非叶子节点的下标一定是<=n/2 - 1;(问就是别人找到的规律)
        siftDown(nums, nums.size(), i); //由顶至底执行堆化
    }
    //从大顶堆中弹出元素
    for (int i = nums.size()-1; i > 0; --i) {
        swap(nums[0], nums[i]); //交换根节点和最右叶子节点
        siftDown(nums, i, 0); //以根节点为起点，从顶至底执行堆化
    } 
}
```

- 时间复杂度：$O(n\log n)$ <!-- 从堆中提取最大元素要n-1次，而且提取完之后堆化是logn -->
- 空间复杂度：$O(1)$
- 不稳定排序



==思考：建堆操作为啥时间复杂度是$O(n)$？==



### **桶排序**

```c++
/* 桶排序 */
void bucketSort(vector<float>& nums) {
    int k = nums.size() / 2; //初始化k = n/2个桶，预期向每个桶分配2个元素
    vector<vector<float>> buckets(k);
    for (float num: nums) { //1. 将数组元素分配到各个桶中
        // 输入数据范围为[0, 1)，使用num * k映射到索引范围[0, k-1]
        int i = num * k;
        buckets[i].push_back(num);
    }
    //2. 对各个桶执行排序
    for (vector<float>& bucket: buckets) {
        sort(bucket.begin(), bucket.end()); //默认按照升序对桶内元素进行排序
    }
    //3. 遍历桶，合并结果
    int i = 0;
    for (vector<float>& bucket: buckets) {
        for (float num: bucket) {
            num[i] = num;
        	++i;
        }
    }
}
```

- 时间复杂度：$O(n+k)$ <!-- 分配元素到桶O(n)，合并桶O(k) -->
- 空间复杂度：$O(n+k)$ <!-- k个桶，n个额外空间 -->
- 桶排序是否稳定取决于排序桶的算法是否稳定



### **计数排序**

```c++
/* 计数排序 */
void countingSortNaive(vector<int>& nums) {
    int m = 0;
    for (int num: nums) {
        m = max(num, m); //1. 统计数组最大元素m
    }
    vector<int> counter(m + 1, 0); //2. 统计各数字的出现次数 count[num] 代表 num的出现次数
    for (int num: nums) {
        counter[num]++;
    }
    int i = 0; //3. 遍历counter，将各元素填回原数组
    for (int num = 0; num < m + 1; ++num) {
        for (int j = 0; j < counter[num]; ++j, ++i) {
            nums[i] = num;
        }
    }
}
```

如果输入数据是对象，那么上述步骤3.就失效了。

```c++
/* 计数排序 */
void countingSort(vector<int>& nums) {
    int m = 0;
    for (int num: nums)
       	m = max(num, m); //1.统计数组最大元素m
    vector<int> counter(m + 1, 0);
    for (int num: nums)
        counter[num]++; //2.统计各数字出现的次数
    for (int i = 0; i < m; ++i) {
        counter[i + 1] += counter[i]; //3.求count的前缀和
    }
    int n = nums.size();
    vector<int> res(n);
    for (int i = n - 1; i >= 0; --i) {
        int num = nums[i];
        res[counter[num] - 1] = num; //将num放置到对应索引处
        counter[num]--;
    }
    nums = res; //覆盖原数组
}
```

这是人能想出来的吗？

- 时间复杂度：$O(n+m)$ <!-- 遍历数组和counter -->
- 空间复杂度：$O(n+m)$
- 稳定排序



### **基数排序**

