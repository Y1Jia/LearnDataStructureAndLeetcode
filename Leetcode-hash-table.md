# Leetcode-hash-table

## 1.两数之和

题目描述:给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

```
示例:
给定 nums = [2, 7, 11, 15], target = 9
因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

基本思路:一个动态查找的过程。对于每个数nums[i],我们要寻找剩下的数中是否存在temp (temp=target-nums[i])，最后返回的是下标。这里用哈希表来实现查找过程。

遍历一次sums数组，对于每个sum[i]，求出temp，并在哈希表中查找是否存在temp，如果存在那么输出下标数组(需要malloc)，如果不存在那么就在哈希表中对应位置插入一个新结点(结点内记录了sum[i]和i,ps:i为下标)。

```c
//分离链接散列表
//先用c语言实现自己的哈希表
struct MyNode{
    int data;          //存放数值
    int index;         //用来存放下标
    struct MyNode* next;
};
struct HashTbl{
    int TableSize;
    struct MyNode** TheLists;   //指针的数组
};
int NextPrime(int n)  //求不小于n的一个素数
{
    if(n%2==0) n++;
    int i;
    for(;;n+=2)
    {
        for(i=3;i<sqrt(n);i+=2)
            if(n%i==0) goto out;
        return n;
        out:;
    } 
}
int Hash(int key,int TableSize)   //散列函数
{
    int ret=abs(key)%TableSize;
    return ret;   //使用绝对值函数以防出现负数的情况
}
struct HashTbl* InitializeTable(int TableSize)
{
    struct HashTbl* H;
    int i;
    H=malloc(sizeof(struct HashTbl));
    H->TableSize=NextPrime(TableSize);     //若TableSize不是素数，则把其变为下一个素数
    H->TheLists=malloc(sizeof(struct MyNode*)*H->TableSize);
    for(i=0;i<H->TableSize;i++)             //配置表头
    {
        H->TheLists[i]=malloc(sizeof(struct MyNode));
        H->TheLists[i]->next=NULL;
        H->TheLists[i]->index=-1;
    }
    return H;
}
struct MyNode* Find(int key,struct HashTbl* H)  //找到则返回结点的地址 找不到则返回NULL
{
    struct MyNode* P;
    struct MyNode* L;
    L=H->TheLists[Hash(key,H->TableSize)];
    P=L->next;                     //跳过表头 从第一个元素开始比对
    while(P!=NULL&&P->data!=key)
       P=P->next;
    return P;
}
void Insert(int key,struct HashTbl* H,int Index)
{
    struct MyNode* Pos;
    struct MyNode* L;
    Pos=Find(key,H);
    if(Pos==NULL)   /*key is not found*/
    {
        struct MyNode* NewCell=(struct MyNode*)malloc(sizeof(struct MyNode));
        L=H->TheLists[Hash(key,H->TableSize)];
        NewCell->next=L->next;            //头插法把新结点插在前面
        L->next=NewCell;
        NewCell->data=key;
        NewCell->index=Index;
    }
}
void FreeHashTable(struct HashTbl* H,int TableSize)
{
    int i=0;
    struct MyNode* temp=NULL,*deleteNode=NULL;
    for(i=0;i<TableSize;i++)
    {
        temp=H->TheLists[i];
        while(temp)
        {
            deleteNode=temp;
            temp=temp->next;
            free(deleteNode);
        }
    }
    free(H);
}
//基本思路:遍历数组，对于每次的值nums[i],可以得到temp=target-nums[i],然后去哈希表中查找是否存在temp，若存在则输出两个数的下标，若不存在就把nums[i]插入到哈希表中
int* twoSum(int* nums, int numsSize, int target, int* returnSize){
    struct HashTbl* HashforIndex=InitializeTable(numsSize);      
    int temp,i;
    *returnSize=0;
    int* output=NULL;
    for(i=0;i<numsSize;i++)
    {
        temp=target-nums[i];
        struct MyNode* findNode=Find(temp,HashforIndex);
        if(findNode!=NULL)   //找到了temp
        {
            output=(int*)malloc(2*sizeof(int));
            *returnSize=2;
            output[0]=findNode->index;
            output[1]=i;
            return output;
        }
        else              //没找到temp
           Insert(nums[i],HashforIndex,i);
    }
    FreeHashTable(HashforIndex,HashforIndex->TableSize);
    return output;
}
```

## 217.存在重复元素

题目描述:给定一个整数数组，判断是否存在重复元素。

如果任意一值在数组中出现至少两次，函数返回 `true` 。如果数组中每个元素都不相同，则返回 `false` 。

和上一题基本相同(不同只在MyNode没有index)。

```c
struct MyNode{
    int data;
    struct MyNode* next;
};
struct HashTbl{
    int TableSize;
    struct MyNode** TheLists;
};
int NextPrime(int n)
{
    if(n%2==0) n++;
    int i;
    for(;;n+=2)
    {
       for(i=3;i<sqrt(n);i++)
          if(n%i==0) goto out;
        return n;
        out:;
    }
}
int Hash(int key,int TableSize)
{
    int ret=abs(key)%TableSize;
    return ret;
}
struct HashTble* InitializeTable(int TableSize)
{
    struct HashTbl* H;
    int i;
    H=malloc(sizeof(struct HashTbl));
    H->TableSize=NextPrime(TableSize);
    H->TheLists=malloc(sizeof(struct MyNode*)*H->TableSize);
    for(i=0;i<H->TableSize;i++)
    {
        H->TheLists[i]=malloc(sizeof(struct MyNode));
        H->TheLists[i]->next=NULL;
    }
    return H;
}
struct MyNode* Find(int key,struct HashTbl *H)
{
    struct MyNode* P,*L;
    L=H->TheLists[Hash(key,H->TableSize)];
    P=L->next;
    while(P!=NULL&&P->data!=key)
       P=P->next;
    return P;
}
void Insert(int key,struct HashTbl *H)
{
    struct MyNode* Pos,*L;
    Pos=Find(key,H);
    if(Pos==NULL)  /*key is not found*/
    {
        struct MyNode* NewCell=(struct MyNode*)malloc(sizeof(struct MyNode));
        L=H->TheLists[Hash(key,H->TableSize)];
        NewCell->next=L->next;
        L->next=NewCell;
        NewCell->data=key;
    }
}
void FreeHashTable(struct HashTbl* H,int TableSize)
{
    int i=0;
    struct MyNode* temp=NULL,*deleteNode=NULL;
    for(i=0;i<TableSize;i++)
    {
        temp=H->TheLists[i];
        while(temp)
        {
            deleteNode=temp;
            temp=temp->next;
            free(deleteNode);
        }
    }
    free(H);
}
bool containsDuplicate(int* nums, int numsSize){
    struct HashTbl *HashForNums=InitializeTable(numsSize+1);
    for(int i=0;i<numsSize;i++)
    {
        struct MyNode* findNode=Find(nums[i],HashForNums);
        if(findNode!=NULL) return true;
        else  Insert(nums[i],HashForNums);
    }
    return false;
}
```

## 594.最长和谐子序列

题目描述:和谐数组是指一个数组里元素的最大值和最小值之间的差别正好是1。

现在，给定一个整数数组，你需要在所有可能的子序列中找到最长的和谐子序列的长度。

```c
//基本思路是通过哈希表实现动态查找。
//在哈希表的结点中记录 数值和出现次数
//想法一:遍历一次数组，把数值和出现次数保存在哈希表中；然后遍历一次哈希表，对于遍历到的每个x，寻找x+1对应的次数，两个次数相加与最大次数比较
//想法二:同样遍历一次数组，把数值和出现次数保存在哈希表中，但每次遍历后，对于当前x，找出x-1和x+1对应的次数，然后x和x-1次数相加，x和x+1次数相加，与当前最大次数比较。
struct MyNode{
    int data;             //记录值
    int index;           //记录次数
    struct MyNode* next;
};
struct HashTbl{
    int TableSize;
    struct MyNode** TheLists;
};
int NextPrime(int n)            //返回不小于n的最小素数
{
    if(n%2==0) n++;
    int i;
    for(;;n+=2)
    {
        for(i=3;i<sqrt(n);i++)
           if(n%i==0) goto out;
        return n;
        out:;
    }
}
int Hash(int key,int TableSize)      //散列函数
{
    int ret=abs(key)%TableSize;
    return ret;
}
struct HashTbl* InitializeHashTbl(int TableSize)
{
    struct HashTbl* H=(struct HashTbl*)malloc(sizeof(struct HashTbl));
    H->TableSize=NextPrime(TableSize);
    H->TheLists=malloc(sizeof(struct MyNode*)*H->TableSize);
    for(int i=0;i<H->TableSize;i++)
    {
        H->TheLists[i]=malloc(sizeof(struct MyNode));
        H->TheLists[i]->next=NULL;
        H->TheLists[i]->index=0;
    }
    return H;
}
struct MyNode* Find(int key,struct HashTbl* H)
{
    struct MyNode* p,*l;
    l=H->TheLists[Hash(key,H->TableSize)];;
    p=l->next;
    while(p!=NULL&&p->data!=key)
       p=p->next;
    return p;       
}
void Insert(int key,struct HashTbl* H)
{
    struct MyNode* Pos,*l;
    Pos=Find(key,H);
    if(Pos==NULL)
    {
        struct MyNode* NewCell=(struct MyNode*)malloc(sizeof(struct MyNode));
        l=H->TheLists[Hash(key,H->TableSize)];
        NewCell->next=l->next;
        l->next=NewCell;
        NewCell->data=key;
        NewCell->index=1;
    }
}
void FreeHashTbl(struct HashTbl* H)
{
    struct MyNode* temp=NULL,*deleteNode=NULL;
    for(int i=0;i<H->TableSize;i++)
    {
        temp=H->TheLists[i];
        while(temp)
        {
            deleteNode=temp;
            temp=temp->next;
            free(deleteNode);
        }
    }
    free(H);
}
int findLHS(int* nums, int numsSize){
    struct HashTbl* HashForNums=InitializeHashTbl(numsSize);
    int max=0;
    for(int i=0;i<numsSize;i++)
    {
        int sum1,sum2;                           //sum1是x-1和x的次数之和  sum2是x和x+1的次数之和
        struct MyNode* findNode=Find(nums[i],HashForNums);
        if(findNode==NULL)                         //若nums[i]不存在则插入，存在则把其次数加一
        {
            Insert(nums[i],HashForNums);   
            sum1=sum2=1;
        } 
        else
        {
            findNode->index++;
            sum1=sum2=findNode->index;     
        }             
        findNode=Find(nums[i]-1,HashForNums);
        if(findNode!=NULL)     sum1 += findNode->index;
        else   sum1=0;                        //题目要求最大值和最小值之间的差别刚好是2，那么意味着x和x-1两个数都得有，若一个为0那么结果就不符合题意
        findNode=Find(nums[i]+1,HashForNums);
        if(findNode!=NULL)     sum2 += findNode->index;
        else   sum2=0;
        int tempsum=fmax(sum1,sum2);
        max=fmax(max,tempsum);
    }
    return max;
}
```

