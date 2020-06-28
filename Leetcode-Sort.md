# Leetcode-Sort

## 堆排序

给定一个数组，若按小到大排序。可使用堆排序。分为两步:①BuildHeap 把数组构建成一个max堆 ；②DeleteMax 在这个过程中，把最后一个元素和第一个元素交换位置，然后把堆的大小减一(即最后一个元素就视为不在堆的内部了)，然后把第一个元素下滤，这样剩下的元素依旧是一个max堆，但是原最大的元素已经位于数组的最后一个位置。重复这个过程就可以实现排序。

```c
#define LeftChild(i) (2*(i)-1)   //i结点的左儿子的下标 (这样计算是因为数组下标是从0开始而不是1开始)
void PercDown(ElementType A[], int i, int N)
{
    int Child;
    ELementType tmp;
    for(tmp=A[]; LeftChild(i)<N; i=Child)
    {
        Child=LeftChild(i);
        if(Child!=N-1 && A[Child+1]>A[Child])
            Child++;
        if(tmp<A[Child])
            A[i]=A[Child];
        else 
            break;
    }
    A[i]=tmp;
}
void Heapsort(ElementType A[],int N)
{
    int i;
    for(i=N/2; i>=0; i--)   /*BuildHeap*/
        PercDown(A,i,N);
    for(i=N-1;i>0; i--)     /*DeleteMax*/
    {
        Swap(&A[0],&A[i]); 
        PercDown(A,0,i);
    }
}
```



### 215.数组中的第k个最大元素

```c
//使用堆排序，先把所给的数组构建成一个max堆，然后用k次的deleteMax来取出第k个最大的元素
#define LeftChild(i) (2*(i)+1) //i结点的左儿子的下标
void PercDown(int* nums,int i,int N);
void BuildHeap(int* nums, int N);
void MydeleteMaxForK(int* nums,int k,int N);
void Swap(int* i,int *j);
int findKthLargest(int* nums, int numsSize, int k){
    BuildHeap(nums,numsSize);
    MydeleteMaxForK(nums,k,numsSize);
    return nums[numsSize-k];
}
//max堆的下滤函数 要用于BuildHeap 和 deleteMax
void PercDown(int* nums,int i,int N)  //i为当前要下滤的结点  N为总的结点个数
{
    int Child,tmp;
    for(tmp=nums[i]; LeftChild(i)<N;i=Child)  //LeftChild(i)<N 确保了该节点不是叶结点(即至少有左儿子)
    {
        Child=LeftChild(i);
        if(Child!=N-1 &&nums[Child+1]>nums[Child]) //这里的顺序不能换 第一个表达式判断下一个结点是否存在(即父节点的右儿子) 
           Child++;       // 然后找出父节点的左右儿子中较大的那个(即 如果右儿子比左儿子大 就把Child++)
        if(tmp<nums[Child])    //如果要下滤的结点小的话就把 Child的结点向上移一个位置
           nums[i]=nums[Child];
        else 
           break;   //如果下滤的结点比Child大 那么下滤过程结束  break出去然后把当前的结点赋值tmp        
    }
    nums[i]=tmp;
}
void BuildHeap(int* nums, int N)   //创建max堆
{
    int i;
    for(i=N/2; i>=0; i--)   //从最后一个结点的父节点开始到第一个结点  逐个调用下滤函数 即构建了一个
       PercDown(nums,i,N);
}
void MydeleteMaxForK(int* nums,int k,int N)  //这里把最大元素放到堆的最后一个位置，然后把堆的大小减一，那么执行k次后，堆的倒数第k个位置的元素就是第k个最大元素
{
    int i;
    for(i=N-1; i>0&&k>0; i--,k-- )
    {
        Swap(&nums[0],&nums[i]);  //把最大的元素和最后一个元素交换位置
        PercDown(nums,0,i);   //i每次减一 实现了在每次下滤时 堆的大小减一
    }
}
void Swap(int* i,int *j)
{
    int temp=*i;
    *i=*j;
    *j=temp;
}
```

