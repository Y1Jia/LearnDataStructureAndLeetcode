# Leetcode 树专题

## 递归

一棵树要么是空树，要么有两个指针，每个指针指向一棵树。树是一种递归结构，很多树的问题可以使用递归来处理。

### 104.二叉树的最大深度

二叉树的最大深度就是它的根节点的 左右子树深度最大的那个加一 

```c
#include<math.h>
int maxDepth(struct TreeNode* root){
    if(root==NULL) return 0;
    return fmax(maxDepth(root->left),maxDepth(root->right))+1;
}
```

### 110.平衡二叉树

#### 方法一:自顶向下递归***

利用height函数计算每个结点的高度。接下来就是比较每个节点左右子树的高度。在一棵以 r为根节点的树T 中，只有每个节点左右子树高度差不大于1 时，该树才是平衡的。因此可以比较每个节点左右两棵子树的高度差，然后向上递归。

在这个算法中，isBlance函数会遍历每个结点，调用height函数，而调用一次height函数就会遍历当前结点以及其子树，这样会产生很多的重复计算量。

```c
 //自顶向下的递归，从根节点向叶结点遍历(先做左子树再右子树)   比较各个结点左右子数的高度差(比较是从底向上的)
#include<math.h>
bool isBalanced(struct TreeNode* root){
    if(root==NULL)  return true;
    return abs(height(root->left)-height(root->right))<2 && isBalanced(root->left) && isBalanced(root->right);
}
int height(struct TreeNode* root){           //计算每个节点的深度
    if(root==NULL) return-1;                 //注意是-1而不是0  因为若到了叶结点应该return 0,而下一行表达式里面有一个+1
    return 1+fmax(height(root->left),height(root->right));
}
```

#### 方法二:自底向上递归***

方法一自顶向下，求height时会有很多的重复运算，因此应该想办法遍历一次树就得到答案，即在左右子树不平衡的时候提前结束

而在方法二中，isBlance函数只调用一次height函数，在height函数中会遍历树一次，在计算过程中，同时比较左右子树深度差，因此只用遍历一次树。同时通过“剪枝”的手段，减少运算量。

此处的“剪枝”:用全局变量result，在遇到左右子树不平衡时直接返回0 (因为运用了全局变量，可能leetcode中会无法通过)

```c
bool result=true;
bool isBalanced(struct TreeNode* root){
    height(root);
    return result;
}
int height(struct TreeNode* root){
    if(result==false) return 0;        //若结果已经确定不是平衡二叉树 则逐层返回，得以减少运算量
    if(root==NULL)  return 0;
    int l=height(root->left);
    int r=height(root->right);
    if(abs(l-r)>1)  result=false;
    return 1+fmax(l,r);
}
```

### 543.二叉树的直径

给定一棵二叉树，你需要计算它的直径长度。一棵二叉树的直径长度是任意两个结点路径长度中的最大值。这条路径可能穿过也可能不穿过根结点。示例 :
给定二叉树

          1
         / \
        2   3
       / \     
      4   5    
返回 3, 它的长度是路径 [4,2,1,3] 或者 [5,2,1,3]。

```c
//思路:二叉树的直径即最大的最小路径，这个路径可能不经过根节点，但肯定会经过一个结点(即该路径由一个父结点和它的子树向下延伸组成)
//那么可以递归计算每个结点作为父节点时的路径长度(左子树深度+右子树深度+2)  
//利用全局变量在遍历的过程中记录最长路径 这样可以较少重复计算 最后输出 (类似上题中的自底向上递归的方法)
int max;
int diameterOfBinaryTree(struct TreeNode* root){
    if(root==NULL) return 0;
    max=0;
    depth(root);
    return max;
}
int depth(struct TreeNode* root){     //计算结点深度
    if(root==NULL) return -1;         
    int l=depth(root->left);
    int r=depth(root->right);
    max=fmax(max,l+r+2);           //比较并记录最大的路径长度
    return fmax(l,r)+1;            //返回该结点为根的子数的深度
}
```

### 226.翻转二叉树

```c
//利用递归解决 
//终止条件:结点为NULL
//返回值:已经反转的子树
//每层递归的任务:左右子树交换
struct TreeNode* invertTree(struct TreeNode* root){
    if(root==NULL) return NULL;
    struct TreeNode* temp=root->left;
    root->left=invertTree(root->right);
    root->right=invertTree(temp);
    return root;
}
```

### 617.合并二叉树

```c
//利用递归实现
//终止条件:两个树中有一个结点为NULL(此时另一颗树后面直接接上就行了)
//返回值:已经合并的树的根节点
//每层递归内容:若都不为NULL，数值相加；若有一个为NULL，则取另一个结点为新结点
struct TreeNode* mergeTrees(struct TreeNode* t1, struct TreeNode* t2){
    if(t1==NULL) return t2;        //如果t1,t2都为NULL,则返回的也是NULL
    if(t2==NULL) return t1;
    t1->val += t2->val;
    t1->left=mergeTrees(t1->left,t2->left);
    t1->right=mergeTrees(t1->right,t2->right);      
    return t1;
}
```

### 112.路径总和

```c
//利用递归实现
//终止条件:到达叶结点(左右子树都为空)
//每层递归任务；如果不是叶结点则调用其左右子树的函数(其中参数sum要减去该节点的值)，若是叶结点，那么判断sum是否和叶结点的值相等，返回结果
bool hasPathSum(struct TreeNode* root, int sum){
    if(root==NULL) return false;
    if(root->left==NULL&&root->right==NULL) return sum==root->val;
    else return hasPathSum(root->left,sum-root->val)||hasPathSum(root->right,sum-root->val);  //*重点 每一步减去一个父节点的值，这样到最后的叶结点便容易判断结果
}
```

### 437.路经总和III

题目描述：给定一个二叉树，它的每个结点都存放着一个整数值。

找出路径和等于给定数值的路径总数。

路径不需要从根节点开始，也不需要在叶子节点结束，但是路径方向必须是向下的（只能从父节点到子节点）。

二叉树不超过1000个节点，且节点数值范围是 [-1000000,1000000] 的整数。

示例：

    root = [10,5,-3,3,2,null,11,3,-2,null,1], sum = 8
          10
         /  \
        5   -3
       / \    \
      3   2   11
     / \   \
    3  -2   1
    返回 3。和等于 8 的路径有:
    1.  5 -> 3
    2.  5 -> 2 -> 1
    3.  -3 -> 11

```c
//思路利用两个递归
//第一个递归遍历所有结点
//第二个递归从遍历到的结点向下遍历 判断是否有和为sum的情况
//利用一个int整型ret记录次数  若有和为sum则ret++
int pathSumStartWithRoot(struct TreeNode* root,int sum);
int pathSum(struct TreeNode* root, int sum){
    if(root==NULL) return 0;
    int ret=pathSumStartWithRoot(root,sum)+pathSum(root->left,sum)+pathSum(root->right,sum);
    return ret;
}
int pathSumStartWithRoot(struct TreeNode* root,int sum){
    if(root==NULL) return 0;
    int ret=0;
    if(root->val==sum) ret++;
    ret += pathSumStartWithRoot(root->left,sum-root->val)+pathSumStartWithRoot(root->right,sum-root->val);
    return ret;
}
```

### 572.另一个树的子树

题目描述:给定两个非空二叉树 s 和 t，检验 s 中是否包含和 t 具有相同结构和节点值的子树。s 的一个子树包括 s 的一个节点和这个节点的所有子孙。s 也可以看做它自身的一棵子树。

    示例 1:              
    给定的树 s:                          
         3
        / \
       4   5
      / \
     1   2
    给定的树 t：
       4 
      / \
     1   2
     返回 true，因为 t 与 s 的一个子树拥有相同的结构和节点值。
    示例 2:
    给定的树 s：
         3
        / \
       4   5
      / \
     1   2
        /
       0
    给定的树 t：
       4
      / \
     1   2
    返回 false。
```c
//利用两个递归实现
//第一个递归遍历所有结点
//第二个递归判断从某个节点开始的子树是否与所给的子树相同
bool isSubtreeStartWithRoot(struct TreeNode* s,struct TreeNode* t);
bool isSubtree(struct TreeNode* s, struct TreeNode* t){
    if(s==NULL) return false;
    return isSubtreeStartWithRoot(s,t)||isSubtree(s->left,t)||isSubtree(s->right,t);
}
bool isSubtreeStartWithRoot(struct TreeNode* s,struct TreeNode* t){
    if(t==NULL && s==NULL) return true;            //若包含这个子树，那么t和s应最后同时到NULL
    if(t==NULL || s==NULL) return false;
    if(s->val==t->val) return isSubtreeStartWithRoot(s->left,t->left)&&isSubtreeStartWithRoot(s->right,t->right);
    else return false;
}
```

### 101.对称二叉树

题目描述:给定一个二叉树，检查它是否是镜像对称的。

```c
//利用递归实现
//使用同步移动的两个指针p,q 从根节点开始p q向相反方向移动(p左q右， p右q左)
//比较p,q两个结点的值是否相同
bool check(struct TreeNode* p,struct TreeNode* q);
bool isSymmetric(struct TreeNode* root){
    return check(root,root);
}
bool check(struct TreeNode* p,struct TreeNode* q){
    if(!p && !q) return true;
    if(!p || !q) return false;
    return p->val==q->val && check(p->left,q->right) && check(p->right,q->left);
}
```

### 111.二叉树的深度

给定一个二叉树，找出其最小深度。

最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

**说明:** 叶子节点是指没有子节点的节点。

```c
//注意点:递归的终止条件是到叶子节点，即左右子树均为空
//1因此当左右子树均为空时，返回1
//2当左右子树有一个为空时，返回非空子树的深度(注意+1)
//3当左右子树均不空时，返回左右子树深度中小的那个(注意+1)
int minDepth(struct TreeNode* root){
    if(root==NULL) return 0;
    int l=minDepth(root->left);
    int r=minDepth(root->right);
    if(root->left==NULL || root->right==NULL) return l+r+1;   //1 2被合并为这个语句
    return fmin(l,r)+1;                                       //3
}
```

### 404.左子叶之和

计算给定二叉树的所有左叶子之和。

```c
//递归遍历每一个结点，判断每个结点的左儿子是不是叶子，如果是，则加上
//对于每个结点，如果其左儿子是叶子结点，那么其左儿子的值应该叫上，并继续递归调用判断当前结点的右儿子
//如果当前结点的左儿子不是叶子结点，那么递归调用判断当前节点的左儿子和右儿子
bool IsLeaf(struct TreeNode* root);
int sumOfLeftLeaves(struct TreeNode* root){
    if(root==NULL) return 0;
    if(IsLeaf(root->left)) return root->left->val + sumOfLeftLeaves(root->right);
    else return sumOfLeftLeaves(root->left)+sumOfLeftLeaves(root->right);
} 
bool IsLeaf(struct TreeNode* root){
    if(root==NULL) return false;
    return root->left==NULL && root->right==NULL;
}
```

### 687.最长同值路径🔺***

```c
//******🔺
//递归遍历每个结点，计算每个结点开始的同路径长度，记录最大值 (自底向上递归)
int dfs(struct TreeNode* root);
int path;
int longestUnivaluePath(struct TreeNode* root){
    path=0;
    dfs(root);
    return path;
}
int dfs(struct TreeNode* root){
    if(root==NULL) return 0;
    int left=dfs(root->left);
    int right=dfs(root->right);
    int leftPath= root->left!=NULL&&root->left->val==root->val? left+1: 0;
    int rightPath= root->right!=NULL&&root->right->val==root->val? right+1: 0;
    path=fmax(path,leftPath+rightPath);
    return fmax(leftPath,rightPath);
}
```

### 337.打家劫舍III🔺***

题目描述:这个地区只有一个入口，我们称之为“根”。 除了“根”之外，每栋房子有且只有一个“父“房子与之相连。一番侦察之后，聪明的小偷意识到“这个地方的所有房屋的排列类似于一棵二叉树”。 如果两个直接相连的房子在同一天晚上被打劫，房屋将自动报警。

计算在不触动警报的情况下，小偷一晚能够盗取的最高金额。

PS:这里好像用到了动态规划？？？？

```c
//此题并不是隔层求解那么简单
//终止条件:结点为NULL
int rob(struct TreeNode* root){
    if(root==NULL) return 0;
    int sum1=root->val;
    if(root->left!=NULL) sum1 += rob(root->left->left)+rob(root->left->right);
    if(root->right!=NULL) sum1 += rob(root->right->left)+rob(root->right->right);
    int sum2=rob(root->left)+rob(root->right);
    return fmax(sum1,sum2);
}
```

### 671.二叉树中第二小的节点

```c
//特点：一个节点要么具有 0 个或 2 个子节点，如果有子节点，那么根节点是最小的节点。
//递归终止条件:到达叶儿子
//返回值:若某一子树的值和根的值都相同，则返回另一子树第二大的值；若左右子树都有比根节点大的值，返回小的那个
//递归任务:比较左右结点和根节点的大小，若相同则继续向下递归
int findSecondMinimumValue(struct TreeNode* root){
    if(root==NULL) return -1;
    if(root->left==NULL&&root->right==NULL) return -1;
    int leftVal=root->left->val;
    int rightVal=root->right->val;
    if(root->val==leftVal) leftVal=findSecondMinimumValue(root->left);
    if(root->val==rightVal) rightVal=findSecondMinimumValue(root->right);
    if(leftVal!=-1&&rightVal!=-1) return fmin(leftVal,rightVal);
    if(leftVal==-1) return rightVal;
    return leftVal;
}
```

## 层次遍历

### 637.二叉树的层平均值

给定一个非空二叉树, 返回一个由每层节点平均值组成的数组.

#### 方法:用队列实现层次遍历***

给了一棵树，把根节点push进去，然后计算count=rear-front，此时count为1，就是当前层的结点个数；每次都这样记录每层的结点数，利用这个count来控制循环次数，循环中把队列结点pop出来，pop出来的即为当前层结点，然后可以进行求和等操作，再把此节点的左右儿子push进队列。

```c
#define Maxsize 10000
typedef struct queue{
    struct TreeNode* node;
    int front;
    int rear;
}MyQueue;
void initQueue(MyQueue* queue){   //初始化队列
    queue->node=(struct TreeNode*)malloc(Maxsize*sizeof(struct TreeNode));
    queue->front=0;
    queue->rear=0;
}
void push(MyQueue* queue,struct TreeNode* root){  //入队列
    queue->node[queue->rear++]=root[0]; //root[0]才可以
}
struct TreeNode* delete(MyQueue* queue){    //出队列
    return &queue->node[queue->front++];  //用&取地址
}
bool isEmpty(MyQueue* queue){       //判断队列是否为空
    if(queue->front==queue->rear) return true;
    return false;
}
//先malloc申请内存(队列头结点和返回的数组)、初始化参数
//把树的根节点放入队列中，然后开始双循环
//外层循环结束条件:队列为空  循环任务:计算这一层结点数，并赋给num,sum初始化， 用内层循环计算sum， 把结果放入数组中
//内层循环结束条件:num为0 循环任务:delete队列，把弹出结点的值加到sum上，并把弹出的结点的左右儿子push进队列(如果有)，同时num--(num为上一层的结点数，num为0，即上一层结点全部弹出，队列中就是当前层的全部结点)
double* averageOfLevels(struct TreeNode* root, int* returnSize){
    MyQueue* queue=(MyQueue*)malloc(sizeof(MyQueue));
    initQueue(queue);
    int count,num;
    double sum;
    double* output=(double*)malloc(Maxsize*sizeof(double));
    *returnSize =0;
    if(root==NULL) return NULL;
    push(queue,root);
    while(!isEmpty(queue))     //外层循环结束条件:数列非空
    {
        count=queue->rear-queue->front;
        num=count;
        sum=0.0;
        while(num>0)      //内存循环结束条件:num>0 (num=0时上一层的结点已经全部出队列了，队列里都是当前层的结点)
        {
            root=delete(queue);
            sum += root->val;
            if(root->left) push(queue,root->left);
            if(root->right) push(queue,root->right);
            num--;
        }
        output[(*returnSize)++]=sum/count;
    }
    return output;
}
```

### 513.找树左下角的值

给定一个二叉树，在树的最后一行找到最左边的值。

思路:这题做法和637类似，同样是利用队列实现树的逐层遍历，但是题目的特殊性是找最后一行最左边的值，那么可以改进下637中的代码，637中每行是从左向右遍历，若改为从右向左遍历，那么结果就是队列最后一个值。(改为从右向左只需把 `if(root->left) push(queue,root->left) `和 `if(root->right) push(queue,root->right) `交换顺序即可)

```c
//定义队列和基本的函数 和637中相同(这里省去了，但是做题时还是得写)
//用链表实现树的逐层遍历，在循环中将每层结点按从右往左的顺序放入链表，这样左下角的结点即为最后一个结点
int findBottomLeftValue(struct TreeNode* root){
    Myqueue* queue=(Myqueue*)malloc(sizeof(Myqueue));
    initQueue(queue);
    if(root==NULL) return NULL;
    push(queue,root);
    while(!isEmpty(queue)){
        root=pop(queue);
        if(root->right) push(queue,root->right);
        if(root->left)  push(queue,root->left);
    }
    return root->val;
}
```

## 前中后序遍历***(重要)

```
    1
   / \
  2   3
 / \   \
4   5   6
```

- 层次遍历顺序：[1 2 3 4 5 6]
- 前序遍历顺序：[1 2 4 5 3 6]   (对于每个结点)访问根节点-左子树-右子树
- 中序遍历顺序：[4 2 5 1 3 6]   访问左子树-根节点-右子树
- 后序遍历顺序：[4 5 2 6 3 1]   访问左子树-右子树-根节点

层次遍历使用 BFS 实现，利用的就是 BFS 一层一层遍历的特性；而前序、中序、后序遍历利用了 DFS 实现。

前序、中序、后序遍只是在对节点访问的顺序有一点不同，其它都相同。

递归的方法如下

① 前序

```c
void dfs(TreeNode root) {
    visit(root);
    dfs(root.left);
    dfs(root.right);
}
```

② 中序

```c
void dfs(TreeNode root) {
    dfs(root.left);
    visit(root);
    dfs(root.right);
}
```

③ 后序

```c
void dfs(TreeNode root) {
    dfs(root.left);
    dfs(root.right);
    visit(root);
}
```

### 114.非递归实现二叉树的前序遍历

给定一个二叉树，返回它的 *前序* 遍历。

 **示例:**

```
输入: [1,null,2,3]  
   1
    \
     2
    /
   3 

输出: [1,2,3]
```

```c
 //迭代实现二叉树前序遍历
 //用栈实现二叉树的前序遍历 在push时先push右子树再push左子树 以保证先访问左子树
 //利用数组模拟栈 以及 记录访问的结点的值
#define Maxsize 1000
int* preorderTraversal(struct TreeNode* root, int* returnSize){
    *returnSize=0;             //要在return NULL 之前给 *returnSize先赋值，否则root==NULL时会报错
    if(root==NULL) return NULL;
    struct TreeNode** stack=(struct TreeNode**)malloc(Maxsize*sizeof(struct TreeNode*)); //注意申请stack内存的式子
    int* output=(int*)malloc(Maxsize*sizeof(int));
    int top=-1;
    stack[++top]=root;
    while(top!=-1)
    {
        struct TreeNode* node=stack[top--];
        output[(*returnSize)++]=node->val;
        if(node->right) stack[++top]=node->right;   //先push右子树以保证左子树先被弹出
        if(node->left)  stack[++top]=node->left;
    }
    return output;
}
```

### 145.非递归实现二叉树的后序遍历

/前序遍历中顺序为root-left-right,后序遍历顺序为left-right-root,可以把前序遍历的顺序改为root-right-left，那么就和后序遍历刚好相反。(那么把修改后的前序遍历的结果反过来就是后序遍历的结果)

```c
//用数组模拟栈 对每个结点的子结点，先push左子树再push右子树
//用链表来实现输出 可以一个个插入后反转 或者 使用头插法(这里使用的是头插法)
//最后把链表中的值移回到数组中，返回数组(若返回链表，和函数要求的int*不符)
#define MaxSize 1000
typedef struct LinkList{
    int data;
    struct LinkList* next;
}MyList;
void addFirst(MyList* l,int x){  //链表头插法函数
    MyList* temp=(MyList*)malloc(sizeof(MyList));
    temp->data=x;
    temp->next=l->next;
    l->next=temp;
}
int* postorderTraversal(struct TreeNode* root, int* returnSize){
    *returnSize=0;
    if(root==NULL) return NULL;
    struct TreeNode** stack=(struct TreeNode**)malloc(MaxSize*sizeof(struct TreeNode*));
    int top=-1;
    MyList* output=(MyList*)malloc(sizeof(MyList));
    output->data=0;output->next=NULL;
    stack[++top]=root;
    while(top!=-1)
    {
        struct TreeNode* temp=stack[top--];
        addFirst(output,temp->val);        //使用头插法实现倒序
        (*returnSize)++;
        if(temp->left) stack[++top]=temp->left;
        if(temp->right) stack[++top]=temp->right;
    }
    int* result=(int*)malloc((*returnSize)*sizeof(int));   //因为函数要返回的是 int*类型 不能直接返回链表 那么把链表的值移到数组中去，返回数组  (这里对数组的申请应用malloc，不然可能会报错“load of null pointer of type 'int”)
    MyList* temp=output->next;
    for(int i=0;i<*returnSize;i++)
    {
        result[i]=temp->data;
        temp=temp->next;
    }
     return result;
}
```

### 94.非递归实现二叉树的中序遍历

```c
//用栈实现中序遍历
//对于每个结点 通过while循环把它的左儿子压入栈中，直至左儿子为NULL  然后在pop，记录值，并把指针移到右儿子开始继续上面的循环
#define Maxsize 1000
int* inorderTraversal(struct TreeNode* root, int* returnSize){
    *returnSize=0;
    if(root==NULL) return NULL;
    struct TreeNode** stack=(struct TreeNode**)malloc(Maxsize*sizeof(struct TreeNode*));
    int top=-1;
    int* output=(int*)malloc(Maxsize*sizeof(int));
    struct TreeNode* cur=root;
    while(cur!=NULL || top!=-1)  //若栈空了且当前节点为NULL 则遍历结束
    {
        while(cur!=NULL)       //沿着左儿子的路径一路push 保证了对于每棵子树最先访问的是左儿子
        {
            stack[++top]=cur;
            cur=cur->left;
        }
        cur=stack[top--];      //对于取出来的结点来说，它的左子树要么为空，要么已经访问过了(因为栈后进先出的原理，它的左儿子们都已经pop出来了);因此，此时访问该节点(即这棵子树的根节点)，然后再把指针移向右子树
        output[(*returnSize)++]=cur->val;
        cur=cur->right;
    }
    return output;
}
```

## 二叉查找树(BST)

### 669.修剪二叉搜索树

给定一个二叉搜索树，同时给定最小边界L 和最大边界 R。通过修剪二叉搜索树，使得所有节点的值在[L, R]中 (R>=L) 。你可能需要改变树的根节点，所以结果应当返回修剪好的二叉搜索树的新的根节点。

```
Input:
    3
   / \
  0   4
   \
    2
   /
  1
  L = 1
  R = 3
Output:
      3
     /
   2
  /
 1
```

```c
//注意示例二，当前结点可能越界了，但是它下面的孩子结点不一定越界
//使用递归求解
//递归结束条件:结点为空或结点的值超过了边界
//返回值:返回修剪过的二叉搜索树的结点 若结点为空则返回NULL，若当前节点比最小界小，则返回右子树的递归；若当前结点比最大界大，则返回左子树的递归;如果当前节点未超过边界值，则左右子树进行递归
struct TreeNode* trimBST(struct TreeNode* root, int L, int R){
    if(root==NULL) return NULL;
    if(root->val<L) return trimBST(root->right,L,R);
    if(root->val>R) return trimBST(root->left,L,R);
    root->left=trimBST(root->left,L,R);
    root->right=trimBST(root->right,L,R);
    return root;
}
```

### 230.二叉搜索树中第k小的元素

#### 方法一(递归)

通过递归实现中序遍历，把遍历结果存在数组中，然后输出数组第k个元素(下标为k-1)

```c
///////////////////////////////////////////第一次自我尝试时的做法
//通过中序遍历找第k小的元素
//递归法  利用BST中序遍历结果为升序的特点
//中序遍历全部元素存在数组中然后返回第k个
int* search(struct TreeNode* root,int* ret,int* num);
#define Maxsize 10000
int kthSmallest(struct TreeNode* root, int k){
    int* ret=(int*)malloc(Maxsize*sizeof(int));       
    int num=0;
    ret=search(root,ret,&num);
    return ret[k-1];
}
int* search(struct TreeNode* root,int* ret,int* num){  //中序遍历并返回结果数组
    if(root==NULL) return NULL;
    search(root->left,ret,num);
    ret[(*num)++]=root->val;
    search(root->right,ret,num);
    return ret;
}
```

#### 方法二(迭代，栈)

迭代。利用栈实现中序遍历，pop时按照升序pop，因此把k作为计数器，每次pop时k-1,当k为0时，当前pop出来的结点的值就是第k小的值，输出即可。

```c
//方法二 参考94题非递归实现二叉树的中序遍历
//迭代 用栈实现二叉树的中序遍历
#define Maxsize 10000
int kthSmallest(struct TreeNode* root, int k){
    if(root==NULL) return NULL;
    struct TreeNode** stack=(struct TreeNode**)malloc(Maxsize*sizeof(struct TreeNode*));
    int top=-1;
    struct TreeNode* cur=root;
    while(cur!=NULL || top!=-1)
    {
        while(cur!=NULL)
        {
            stack[++top]=cur;
            cur=cur->left;        
        }
        cur=stack[top--];
        if(--k==0) return cur->val;
        cur=cur->right;
    }
    return NULL;
}
```

### 538.把二叉搜索树转换为累加树

#### 方法一(迭代，栈)

```c
//反序中序遍历
//用栈从大到小pop (push时先右再左 “中序遍历的反序”)
//第一次pop时值最大的结点，那么把它加在sum上 此时sum的值就是叠加树该节点的值
//第二次pop时 是值第二大的结点 把它的值加到sum上 那么sum就是最大的值和第二大的值的和 即为叠加树中该结点的值
#define Maxsize 1000
struct TreeNode* convertBST(struct TreeNode* root){
    if(root==NULL) return NULL;
    struct TreeNode** stack=(struct TreeNode**)malloc(Maxsize*sizeof(struct TreeNode*));
    int top=-1;
    int sum=0;
    struct TreeNode* cur=root;
    while(top!=-1 || cur!=NULL)
    {
        while(cur!=NULL)
        {
            stack[++top]=cur;
            cur=cur->right;
        }
        cur=stack[top--];
        sum += cur->val; //sum中记录 当前节点以及比当前结点的值大的所有节点 的值的总和 (即累加树中这个结点的值)
        cur->val =sum;
        cur=cur->left;
    }
    return root;
}
```

#### 方法二(递归)

```c
//方法二递归实现
//其实一个函数就可以实现，但是全局变量sum会出错，所以写了两个函数
struct TreeNode* convert(struct TreeNode* root);
int sum=0;
struct TreeNode* convertBST(struct TreeNode* root){
    sum=0;
    root=convert(root);
    return root;
}
struct TreeNode* convert(struct TreeNode* root){   //递归进行反序中序遍历修改结点的值
    if(root==NULL) return NULL;
    root->right=convert(root->right);
    sum += root->val;
    root->val =sum;
    root->left=convert(root->left);
    return root;
}
```

### 235.二叉树搜索树的最近公共祖先

给定一个二叉搜索树, 找到该树中两个指定节点的最近公共祖先。

百度百科中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（一个节点也可以是它自己的祖先）。”

```c
//用递归实现
//把一个大的问题分解为若干个相同的子问题
//设当前结点为根节点(可能是某子数的根节点)有三种情况:
//1.两个结点分别在左右子树中，那么公共结点就是当前的根节点
//2.两节点中一个结点为根节点，那么公共节点就是当前根节点
//3.两个结点均在左子树(或右子树)，那么递归找左子树(或右子数)即可
//三种情况合并一下即为下面的代码
struct TreeNode* lowestCommonAncestor(struct TreeNode* root, struct TreeNode* p, struct TreeNode* q) {
    if(p->val<root->val && q->val<root->val) return lowestCommonAncestor(root->left,p,q);
    if(p->val>root->val && q->val>root->val) return lowestCommonAncestor(root->right,p,q);
    return root;
}

//方法二迭代，基本思想一样,只不过用指针和循环代替了递归而已
struct TreeNode* lowestCommonAncestor(struct TreeNode* root, struct TreeNode* p, struct TreeNode* q){
    struct TreeNode* cur=root;
    while(cur!=NULL)
    {
        if(p->val<cur->val && q->val<cur->val) cur=cur->left;
        else if(p->val>cur->val && q->val>cur->val) cur=cur->right;
        else return cur;
    }
    return cur;
}
```

