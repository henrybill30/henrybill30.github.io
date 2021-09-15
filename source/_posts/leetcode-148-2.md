---
title: 链表排序
date: 2021-09-15 10:49:13
tags: [leetcode, 链表]
categories: 算法
mathjax: true
---
# 题目描述
Sort a linked list in $O(n log n)$ time using constant space complexity.
以$O(nlogn)$的时间复杂度和常量级别的空间复杂度对链表进行排序。
# 题目分析
题目为排序问题，排序算法大体可以分为两种：比较排序和非比较排序，其中比较排序的时间复杂度最小不会低于$O(nlogn)$，非比较排序的空间复杂度一般为$O(n)$，时间复杂度可以降低到$O(n)$，所以根据题目我们可以确定这个题目要用到的是比较排序。比较排序主要有：插入排序、归并排序、堆排序、快速排序等等，下面总结四种常见的比较排序算法的代码实现，均以数组为例。
1. 插入排序
 插入排序就像玩扑克牌时整理牌一样，从数组的第2位(索引为1)开始，向前寻找该数字应该在的位置，直到最后一个数字，插入排序的时间复杂度为$O(n^2)$。计算方法是1+...+(n-1)的结果。（自己理解，具体的还得多记）
    ```java
    public void insertionSort(int[] A){
        for(int i=1;i<A.length;i++){
            int key = A[i];
            int j = i - 1;
            while (j>=0 && A[j]>key){
                A[i] = A[j];
                i -= 1;
                j -= 1;
            }
            A[j+1] = key;
        }
    }
    ```
 2. 归并排序
  归并排序的思想是将数组不断分裂，直到数组只剩2个或1个元素，然后再合并，利用了递归的理念。归并排序时间复杂度的递归式为$O(n)=2O(n/2)+T(n)+D(n)$该递归式的结果为$O(nlogn)$，即时间复杂度为$O(nlogn)$。
    ```java
	public void mergeSort(int[] A, int s, int e){
        if(s<e){
            int q = (e-s)/2 + s;
            mergeSort(A, s, q);
            mergeSort(A, q+1, e);
            Merge(A, s, q, e);
        }
    }

    public static void Merge(int[] A, int s, int q, int e){
        int lLength = q - s + 1;
        int rLength = e - q;
        int[] l = new int[lLength+1];
        int[] r = new int[rLength+1];
        for(int i=0;i<lLength;i++){
            l[i] = A[s+i];
        }
        for(int j=0;j<rLength;j++){
            r[j] = A[q+j+1];
        }
        l[lLength] = Integer.MAX_VALUE;
        r[rLength] = Integer.MAX_VALUE;
        int i = 0;
        int j = 0;
        for(int k=s;k<=e;k++){
            if(l[i]<r[j]){
                A[k] = l[i];
                i += 1;
            }else {
                A[k] = r[j];
                j += 1;
            }
        }
    }
    ```
  3. 堆排序
堆排序的思想是通过建立最大堆或最小堆来实现排序，最大堆或最小堆是一种特殊的二叉树，这种二叉树的特点是所有父节点一定大于或小于其所有孩子节点。堆排序的操作顺序为建堆-->输出根节点-->维护最大堆。通过这样的循环操作就可以将数组排序。
```java
public class HeapSort {


    private static int Parent(int i){
        return i/2;
    }

    private static int leftChild(int i){
        return 2*i;
    }

    private static int rightChild(int i){
        return 2*i+1;
    }

    private static void maxHeap(int[] A, int i, int size){
        int l = leftChild(i)-1;
        int r = rightChild(i)-1;
        int largest = i;
        largest = getLargest(A, size, l, largest);
        largest = getLargest(A, size, r, largest);
        if(largest!=i){
            int tmp = A[largest-1];
            A[largest-1] = A[i-1];
            A[i-1] = tmp;
            maxHeap(A, largest, size);
        }
    }

    private static int getLargest(int[] A, int size, int l, int largest) {
        if(l<=size && A[l]>A[largest-1]){
            largest = l + 1;
        }
        return largest;
    }

    private static void buildMaxHeap(int[] A){
        for(int i=A.length/2;i>0;i--){
            maxHeap(A, i, A.length-1);
        }
    }

    public void heapSort(int[] A){
        int size = A.length-1;
        buildMaxHeap(A);
        for(int i=0;i<A.length;i++){
            int tmp = A[0];
            A[0] = A[size];
            A[size] = tmp;
            size -= 1;
            maxHeap(A, 1, size);
        }
    }
}
```
4. 快速排序
快速排序可以说是比较排序算法中时间复杂度最低的算法，相比堆排序，其时间复杂度中的系数更小。快排的思想是通过递归的方式不断找到数组中元素应该在的位置，最核心的算法就是partition方法。快排的时间复杂度为$O(nlogn)$。
```java
public void quickSort(int[] A, int s, int e){
        if(s<e){
            int q = Partition(A, s, e);
            quickSort(A, s, q-1);
            quickSort(A, q+1, e);
        }
    }

    private static int Partition(int[] A, int s, int e){
        int x = A[e];
        int i = s-1;
        for(int j=s;j<e;j++){
            if(A[j]<x){
                i += 1;
                int tmp = A[i];
                A[i] = A[j];
                A[j] = tmp;
            }
        }
        A[e] = A[i+1];
        A[i+1] = x;
        return i+1;
    }
```
# 解题
本题只是将链表换为数组而已，故可以只需将排序算法稍微改变即可，思想是不变的，解题选用归并排序（因为看了评论很多人都选择了归并），与数组不同，需要通过循环找到链表的中间元素并将链表二分。代码为：
```java
class Solution {
    public ListNode sortList(ListNode head) {
        if(head==null || head.next==null){
            return head;
        }
        
        ListNode half = getMidNode(head);
        ListNode l1 = half.next;
        half.next = null;
        
        return Merge(sortList(head), sortList(l1));
    }
    
    private ListNode Merge(ListNode l1, ListNode l2){
        ListNode l = new ListNode(0);
        ListNode res = l;
        while(l1!=null && l2!=null){
            if(l1.val<l2.val){
                l.next = l1;
                l1 = l1.next;
            }else {
                l.next = l2;
                l2 = l2.next;
            }
            l = l.next;
        }
        
        if(l1!=null){
            l.next = l1;
        }else if(l2!=null){
            l.next = l2;
        }
        
        return res.next;
    }
    
    private ListNode getMidNode(ListNode head) {
        ListNode half = head;
        while(head.next!=null && head.next.next!=null){
            half = half.next;
            head = head.next.next;
        }
        return half;
    }
}
```
希望自己以后能补充上其他排序的链表实现。
# 总结
本题涉及的就是最传统的排序算法，让我对排序算法进行了总结。除此以为，通过查看java源码，发现java在Array.sort()中使用的是TimSort排序算法，似乎在JDK7以前使用的归并排序，TimSort是2002年在Python上提出的算法，该算法结合了归并排序和插入排序，规定当数组长度小于64时使用插入，大于时使用归并，并且再归并时是将数组分成一个个run，而不是一个个元素，每个run都是原来就是升序或降序的，这样排序的效率更高。并且TimSort算法也很稳定。