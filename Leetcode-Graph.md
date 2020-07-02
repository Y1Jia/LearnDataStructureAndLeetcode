# Leetcode-Graph

## 目录

[A.拓扑排序](#A)

​		[207.课程表](#A1)

​		[210.课程表II](#A2)

[B.最短路径算法](#B)

​		[单源无权最短路径](#B1)

​		[单源有权最短路径——Dijkstra算法](#B2)

[C.最小生成树](#C)

​		[Prim算法](#C1)

​		[Kruskal算法](#C2)

## <span id="A">拓扑排序</span>

拓扑排序和其他的排序不同，并不是通过“比较大小”来排次序，而是对于有向图中结点的指向与被指向的关系进行排序。对于任意两个结点，如果结点u指向结点w，那么结果中结点u就排在结点w的前面(并不一定紧挨着)。因此拓扑排序的结果常常不是唯一的。

拓扑排序在实践中可用于排课程，因为有些课程需要先修其他课程。

具体的算法思想：基础是DPS,BPS。(如果使用队列，那么就是BPS，如果使用栈，那么就是DPS)。对于一个图，先遍历一次所有结点，找到所有的入度为0的结点，把他们记录在容器中(通常用栈或者队列)。接下来进行循环排序。对于每个入度为0的结点，把它容器中取出并从图上删去。(删去即“去掉这个结点以及和它相连的边”)。因此对应的，被删去的结点所指向的结点入度减一，减一后判断该结点的入度是否为0，若为0，则把它也记录下来。循环直至容器为空。

但要注意的是，若存在环，比如u指向w,w也指向u，那么这两个结点的入度都不可能为0，因此不可能被加入容器，也无法进行拓扑排序。因此在排序过程中，应使用一个计数器，记录从容器中出来的结点的个数，在循环排序结束后把它与输入的结点个数对比。若个数相同则排序成功；若个数不同，那么排序失败，也证明图中存在 环。

### <span id="A1">207.课程表</span>

题目描述：一个课程可能会先修课程，判断给定的先修课程规定是否合法。

本题不需要使用拓扑排序，只需要检测有向图是否存在环即可。

```c
//拓扑排序基本思路如下:
//使用邻接矩阵(二维数组)存储图的信息
//使用一个数组来记录每个结点的入度
//使用栈来存放入度为0的结点
//用counter记录输出的结点数
//1.遍历数组，每个结点的子节点入度+1
//2.把入度为0的结点存放在栈中
//3.取出栈中的结点并输出，同时使其子结点的入度减一，若子节点入度为0，则把子结点压入栈中
//直至栈为空时退出循环。 若输出的结点数小于所给的结点数，那么证明有环存在。
//此题不需要进行排序，只需要判断是否有环
bool canFinish(int numCourses, int** prerequisites, int prerequisitesSize, int* prerequisitesColSize){
    int Graph[numCourses][numCourses];
    memset(Graph,0,sizeof(int)*numCourses*numCourses);
    int stack[numCourses],top=-1;
    int count=0;
    int inDegree[numCourses];
    memset(inDegree,0,sizeof(int)*numCourses);
    for(int i=0;i<prerequisitesSize;i++)      //把图的信息输入邻接矩阵中，同时把每个结点的子节点入度加一
    {
        Graph[ prerequisites[i][1] ][ prerequisites[i][0] ]=1;
        inDegree[ prerequisites[i][0] ] ++;
    }
    for(int i=0;i<numCourses;i++)  //遍历所有结点  把入度为0的结点入栈
       if(inDegree[i]==0)
          stack[++top]=i;
    while(top>=0)
    {
        int tempV=stack[top--];
        count++;
        for(int i=0;i<numCourses;i++)  //对于当前结点，遍历Grap对应的行，如果找到当前节点的子节点，那么把子节点的入度减一
        {
            if(Graph[tempV][i]==1)
            {
                inDegree[i]--;
                if(inDegree[i]==0)
                {
                    stack[++top]=i;                
                }
            }
        }
    }
    if(count==numCourses) return true;
    return false;
}
```

上面的方法用的是邻接矩阵，邻接表的实现如下(来自leetcode用户linge32）

```c
typedef struct node {
    int data;
    struct node *next;
} Node;

typedef struct {
    Node *graph;
} Graph;
    
    Graph *G = (Graph*)malloc(sizeof(Graph));
    G->graph = (Node*)malloc(sizeof(Node) * numCourses);
    for (i = 0; i < numCourses; i++) {
        G->graph[i].data = i;
        G->graph[i].next = NULL;
    }
    /* 维护入度数, 邻接表 */
    for (i = 0; i < prerequisitesSize; i++) {
        degreeIn[prerequisites[i][0]]++; /* 入队 */
        node = (Node*)malloc(sizeof(Node)); /* 链表的头插法 */
        node->data = prerequisites[i][0];
        node->next = G->graph[prerequisites[i][1]].next;
        G->graph[prerequisites[i][1]].next = node;
    }
```

### <span id="A2">210.课程表II</span>

利用拓扑排序排课程。

```c
struct MyNode{
    int data;
    struct node* next;
};
struct Graph{
    struct MyNode* graph;
};
int* findOrder(int numCourses, int** prerequisites, int prerequisitesSize, int* prerequisitesColSize, int* returnSize){
    int* output=(int*)malloc(sizeof(int)*numCourses);
    int count=-1; //记录pop出的结点数 用于最后比较是否有环存在(这里从-1开始 用作输出数组的下标  在最后比较的时候应+1)
    int* inDegree=(int*)malloc(sizeof(int)*numCourses);   //inDegree记录结点的入度
    memset(inDegree,0,sizeof(int)*numCourses);
    int* stack=(int*)malloc(sizeof(int)*numCourses);  //用栈来存放入度为0的结点
    int top=-1;
    /*创建邻接表*/ 
    struct Graph* G=(struct Graph*)malloc(sizeof(struct Graph));
    G->graph=(struct MyNode*)malloc(sizeof(struct MyNode)*numCourses);
    for(int i=0;i<numCourses;i++)
    {
        G->graph[i].data=0;
        G->graph[i].next=NULL;
    }
    for(int i=0;i<prerequisitesSize;i++)
    {
        struct MyNode* tempNode=(struct MyNode*)malloc(sizeof(struct MyNode)); //创建新结点，并用头插法插入邻接表中
        tempNode->data=prerequisites[i][0];
        tempNode->next=G->graph[ prerequisites[i][1] ].next;
        G->graph[ prerequisites[i][1] ].next=tempNode;
        inDegree[ prerequisites[i][0] ]++;   //被指向的那个结点入度加一
    }
    /*先遍历一次所有节点  把入度为零的结点入栈*/
    for(int i=0;i<numCourses;i++)  
       if(inDegree[i]==0)
          stack[++top]=i;
    /*while循环进行排序*/
    while(top!=-1)
    {
        int tempV=stack[top--];
        output[++count]=tempV;
        struct MyNode* tempNode=G->graph[tempV].next;
        while(tempNode!=NULL)         //遍历邻接表对应结点所指向的结点  把他们的入度-1，并把入度为0的结点入栈
        {
            inDegree[tempNode->data]--;
            if(inDegree[tempNode->data]==0)
               stack[++top]=tempNode->data;
            tempNode=tempNode->next;
        }
    }
    if(count+1 == numCourses) *returnSize=numCourses;
    else *returnSize=0;   //若存在环，即不可能完成所有课程，就返回空数组
    /*free前面申请的内存空间*/
    free(stack);
    free(inDegree);
    for(int i=0;i<numCourses;i++)
    {
        struct MyNode* tempNode,*deleteNode;
        tempNode=G->graph[i].next;
        while(tempNode!=NULL)
        {
            deleteNode=tempNode;
            tempNode=tempNode->next;
            free(deleteNode);
        }
    }
    free(G);
    return output;
}
```

## <span id="B">最短路径算法</span>

### <span id="B1">单源无权最短路径</span>

由于无权，求各点最短路径就可直接BFS，从源点开始向外一层层搜索即可。

```c
/*无权最短路径算法的伪代码*/
void Unweighted( Table T)   /*Assume T is initialized*/
{
    Queue Q;
    Vertex V,W;
    Q=CreateQueue( NumVertex );MakeEmpty(Q);
    /*Enqueue the start vertex S, determined elsewhere*/
    Enqueue( S, Q );
    while( !IsEmpty(Q) )
    {
        V=Dequeue( Q );
        T[V].Known =True; //实际过程中这个known域没有使用；因为一个顶点一旦被处理它就不会再次进入队列，也就不需要标记
        for each W adjacent to V  //这一步使用邻接矩阵或者邻接表实现；若使用邻接表，程序的运行时间就是O(|E|+|V|)
            if(T[W].Dist==Infinity)
            {
                T[W].Dist = T[V].Dist+1;
                T[W].Path = V;   //记录S到W的最短路径上，W的前一个结点(即V),由此可逆着退出S到各个顶点的最短路径
                Enqueue( W, Q );
            }
    }
    DisposeQueue(Q); /* Free the memory */
}
```

### <span id="B2">单源有权最短路径——Dijkstra算法</span>

Dijkstra算法是贪婪算法的一个很好的例子。

路径被赋了权值，就不能简单的使用无权时的BFS了，因为赋权的最短路径所经过的边不一定是最少的。

Dijkstra算法的基本思想是:

用一个数组记录每个顶点的最短路径的值(通常被初始化为无穷大),利用一个Known域记录顶点是否已经是最小。

1.前置处理:把源点的路径值输入为0，并把源点的Known设为True;把源点所指向的结点的路径值输入数组(即对应边的权值)。

2.从所有结点中找出最短路径值最小的unkonwn结点，设置为known。

3.对于取出结点V所指向的每个结点W,如果W是unknown的，就比较  T[V].Dist + C<sub>vw</sub> <T[W].Dist ,如果结果为true，那么意味着这条路径比原本W点记录的路经长更短，于是更新 W的最短路径大小 以及 path

4.重复2，3直到所有结点都为known。

```c
/*Dijkstra 算法的声明*/
typedef int Vertex;
struct TableEntry
{
    List Header;  /* Adjacency list*/
    int Known;
    DistType Dist;
    Vertex Path;
};
/*Vertices are numbered from 0*/
#define NotAVertex (-1)
typedef struct TableEntry Table[ NumVertex ];

/*表的初始化例程*/
void InitTable( Vertex Start, Graph G,Table T)
{
    int i;
    ReadGraph(G,T); /*Read graph somehow*/
    for(i=0;i<NumVertex;i++)
    {
        T[i].Known = False;
        T[i].Dist = Infinity;
        T[i].Path = NotAVertex;
    }
    T[Start].Dist=0;
}

/*dijkstra算法的伪代码*/
void Dijkstra(Table T)
{
    Vertex V,W;
    for(;;)
    {
        V = smallest unknown distance vertex;
        if(V==NotAVertex)
            break;
        T[V].Known=True;
        for each W adjacent to V
            if(!T[W].Known)
                if(T[V].Dist + Cvw < T[W].Dist)
                {  /* Update W */
                    Decrease(T[W].Dist to T[V].Dist+Cvw);
                    T[W].Path=V;
                }
    }
}

/*显印实际最短路径的例程*/
/* Print shortest path to V after Dijkstra has run */
/* Assume that the path exists*/
void PrintPath( Vertex V, Table T)
{
    if(T[V].Path!=NotAVertex)
    {
        PrintPath(T[V].path,T);
        printf(" to");
    }
    printf("%v",V);  /* %v is pseudocode(伪码)*/
}
```



## <span id="C">最小生成树</span>

### <span id="C1">Prim算法</span>

### <span id="C2">Kruskal算法</span>



