

# 数据结构

## AVL（平衡二叉树）

平衡二叉树又称 AVL 树，在满足二叉查找树特性的基础上，要求每个节点的左右子树的高度差不能超过 1

当我们插入或删除数据导致不满足平衡二叉树不平衡时，平衡二叉树会进行调整树上的节点来保持平衡

## B树

从磁盘中读取数据时，都是按照磁盘块来读取的，并不是一条一条的读。一个键值对太慢，所以为了更好利用磁盘，为了读到更多的键值。

希望有单个节点可以存储多个键值和数据的平衡树，出现B树。

## B+树

**B+树** 是一个多叉树，也是一个为磁盘而设计的一种平衡查找树，

一个索引一个B+树，非叶子节点存储的是key，叶总节点存储的key和value

主键索引叶子节点： key:主键的值，value:整行数据。 

普通列索引叶子节点： key：索引列的值， value:主键的值。

在 InnoDB 中，我们通过**数据页之间通过双向链表连接**以及**叶子节点中数据之间通过单向链表连接的方式**可以找到表中所有的数据。

为了保持平衡，对于新插入的键值可能需要做大量的拆分页（split）操作，而B+树主要用于磁盘，因此页的拆分意味着磁盘的操作，应该在可能的情况下尽量减少页的拆分。因此，B+树提供了旋转（rotation）的功能。

B+树使用填充因子（fill factor）来控制树的删除变化，50%是填充因子可设的最小值。填充因子小于50%，这时需要做合并操作

## 红黑树





#排序

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200618221536704.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ0ODUwNzI1,size_16,color_FFFFFF,t_70)

## 冒泡排序

1. 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
2. 对第0个到第n-1个数据做同样的工作。这时，最大的数就“冒泡”到了数组最后的位置上。
3. 在未排序的依次类推

## 选择排序

在未排序序列中找到最小（大）元素，存放到排序序列的起始位置

## 插入排序

1. 在未排序序列中找到最小（大）元素，存放到排序序列的起始位置。
2. 再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾
3. 在部分有序的序列中效果奇好

## 希尔排序

插入排序的一种优化，优化插入排序的交换次数过多的问题

用步长来排序，最后是到步长1时插入排序

通常步长的选择是从step=n/2开始，分组，

——8 7 6 5 4  3 2 1

分组8 7 6 5 4  3 2 1

组号0 1 2 3 0 1 2 3

组内插入排序的思想

4 8

3 7

2 6

1 5 

——4 3 2 1 8 7 6 5

每一组内排序，step每次再减半。步长的选择直接决定了希尔排序的复杂度

## 归并排序

归并排序的思想就是先递归分解数组，再合并数组。

时间复杂度（nlogn） 空间复杂度（n）

将已排好序的进行合并成一个新的排序，比如 两个排好序的，将从他们的最左端开始比较合并成新排序，如果是一个没有排序好的序列使用归并排序的话，一般用递归实现，从小到大实现排好序的序列合并新序列。

图片来源：https://www.cnblogs.com/chengxiao/p/6194356.html

代码看算法第四版

[剑指 Offer 51. 数组中的逆序对](https://leetcode-cn.com/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)

```java
public class Solution {
    public int reversePairs(int[] nums) {
        int len=nums.length;
        if(len<2)return 0;
        int[] temp=new int[len];
        int[] copy=new int[len];
        for(int i=0;i<len;i++){
            copy[i]=nums[i];
        }

        return reversePairs(copy,0,len-1,temp);
    }
    public int reversePairs(int[] nums,int left,int right,int[] temp){
        if(left==right){
            return 0;
        }
        int mid=left+(right-left)/2;          //防止溢出；向下取整

        int leftCount=reversePairs(nums,left,mid,temp);
        int rightCount=reversePairs(nums,mid+1,right,temp);
        if(nums[mid]<=nums[mid+1]){
            return leftCount+rightCount;
        }

        int cossCount=mergePairs(nums,left,mid,right,temp);
        

        return leftCount+rightCount+cossCount;
    }
    public int mergePairs(int[] nums,int left,int mid,int right,int[] temp){
        for(int i=left;i<=right;i++){
            temp[i]=nums[i];
        }
        int i=left;
        int j=mid+1;
        int count=0;
        for(int k=left;k<=right;k++){
            if(i==mid+1){
                nums[k]=temp[j];
                j++;
            }else if(j==right+1){
                nums[k]=temp[i];
                i++;
            }else if(temp[i]>temp[j]){//逆序
                nums[k]=temp[j];
                j++;
                count+=mid+1-i;
            }else{
                nums[k]=temp[i];
                i++;
            }
        }
        return count;
    }
}
```

## 快速排序

1. 从数列中挑出一个元素作为基准数。
2. 分区过程，将比基准数大的放到右边，小于或等于它的数都放到左边。
3. 再对左右区间递归执行第二步，直至各区间只有一个数。

```java
import java.util.Arrays;
import java.util.Queue;
import java.util.Scanner;

public class 排序数组 {
    public static void main(String[] args) {
        int []arr = {7,6,7,11,5,12,3,0,1};
        System.out.println("排序前："+ Arrays.toString(arr));
        QuickSort(arr,0,arr.length-1);
        System.out.println("排序前："+Arrays.toString(arr));
    }

    private static void QuickSort(int[] arr,int lo,int hi) {
        if(lo<hi){
            int pos=Parttion(arr,lo,hi);//pos的位置确认了
            QuickSort(arr,lo,pos-1);//递归pos的左未排序区间
            QuickSort(arr,pos+1,hi);//递归pos的右未排序区间
        }
    }

    private static int Parttion(int[] arr, int lo, int hi) {
        int point=arr[lo];//选定一个切入的标准值，先用临时变量存起来，挖坑
        while (lo<hi){
            //右指针扫描如果是大的放在它右边，那么就继续扫描，否则停止
            while(lo<hi&&arr[hi]>=point){
                hi--;
            }
            arr[lo]=arr[hi];//把hi不满足条件的填坑到左边，变成满足条件
            //左指针扫描如果是小的放在它左边，那么就继续扫描，否则停止
            while(lo<hi&&arr[lo]<=point){
                lo++;
            }
            arr[hi]=arr[lo];//把lo不满足条件的填坑到右边，变成满足条件
        }
        arr[lo]=point;//最后arr[lo}这个值必然存储的是某个复制的值，把切入的标准值填入
        return lo;
    }
}

```

## [堆排序](https://blog.csdn.net/qq_36186690/article/details/82505569)

首先数组的序号转化成树，按照完全二叉树的步骤从上到下从左到右填充

<img src="https://images2015.cnblogs.com/blog/1024555/201612/1024555-20161217192038651-934327647.png" alt="img" style="zoom:33%;" />

### 大顶堆

就是父节点大于子节点

从上往下扫描 发现不满足条件的话将子节点中比较大的节点交换

扫描完之后取堆顶元素的话，是把最后一个元素放到上面去

再扫描调整

### 小顶堆

就是父节点小于子节点

![img](https://img-blog.csdn.net/20180908013007479?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTg2Njkw/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```java
import java.util.Arrays;
/**
 * https://blog.csdn.net/qq_44850725/article/details/106820098
 */
public class heapsort {
    public static void main(String []args){
        int []arr = {7,6,7,11,5,12,3,0,1};
        System.out.println("排序前："+Arrays.toString(arr));
        sort(arr);
        System.out.println("排序前："+Arrays.toString(arr));
    }

    public static void sort(int []arr){
        //1.构建大顶堆
        for(int i=arr.length/2-1;i>=0;i--){//这个for循环：从第一个"非叶子结点"从下至上，从右至左调整结构
            
            adjustHeap(arr,i,arr.length);//该例子，进里面的方法发现调用了一次
        }
        //2.调整堆结构+交换堆顶元素与末尾元素
        for(int j=arr.length-1;j>0;j--){
            swap(arr,0,j);//将堆顶元素与末尾元素进行交换
            adjustHeap(arr,0,j);//重新对堆【0，j】进行调整
        }

    }

    /**
     * 调整大顶堆（仅是调整过程，建立在大顶堆已构建的基础上）
     * @param arr
     * @param i
     * @param length
     */
    public static void adjustHeap(int []arr,int i,int length){
        int temp = arr[i];//先取出当前元素i（i=0时 k=1；k<len;k=2*k+1
        for(int k=i*2+1;k<length;k=k*2+1){//从i结点的左子结点2i+1开始
            if(k+1<length && arr[k]<arr[k+1]){//如果左子结点小于右子结点，k指向右子结点
                k++;
            }
            if(arr[k] >temp){//如果k指向的子节点大于父节点，将子节点值赋给父节点（不用进行交换）
                arr[i] = arr[k];
                i = k;
            }else{
                break;
            }
        }
        arr[i] = temp;//将temp值放到最终的位置
    }

    /**
     * 交换元素
     * @param arr
     * @param a
     * @param b
     */
    public static void swap(int []arr,int a ,int b){
        int temp=arr[a];
        arr[a] = arr[b];
        arr[b] = temp;
    }
}

```



# 查找算法

```
//时间复杂度O(logn)
public static int opsForTwoSearch(int num[],int target){
    int left=0;
    int right=num.length-1;
    while(left<=right){
        //[left,right]的中间下标mid
        int mid=left+(right-left)/2;//（1）防止溢出；（2）向下取整
        if(target>num[mid])left=mid+1;
        else if(target<num[mid])right=mid-1;
        else{
            return mid;
        }
    }
    return -1;
}
```

