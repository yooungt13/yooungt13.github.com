---
layout: post
title: RadixSort
tag: DataStructure
addr: Sysu, Guangzhou
---

**本系列所有文章均来自JS大神的日志**　[原文链接](http://user.qzone.qq.com/1033932438/blog/1390204892)

　　基数排序是一种与之前所介绍的排序算法截然不同的排序方法。前面所介绍的排序方法中，都有着共同的一种操作：元素间的比较。而基数排序则不是基于元素间的比较来驱动整个排序过程的，后文将会详细展开介绍。另外，基数排序需要做一些准备工作，以序列中最大者为准，我们需要让其它元素的位数与最大者的位数一致，比它少的则要在数值前补0。这步的目的在于让序列全体元素的位数一致。我们很快会发现，基数排序过程需要操作元素的数位。

　　现在，我们以全新的序列：278、109、63、930、589、184、505、269、8、83为例。开始时，找到该序列最大值930，其是3位数，对序列内剩下的元素，若数位小于3，则全部在其前头补0直到有3位，得：278、109、063、930、589、184、505、008、083。接着，我们发现，所有元素均为10进制数，也就是说元素的每个数位均由0~9这10种数字构成。我们创建10个队列，每个队列刚好对应表示1个十进制数位值，则分别有队列0、队列1、队列2、……、队列9。

　　现在可以正式开始处理了。首先，我们依次看序列所有元素的最低位，并按最低位的数字令该元素进入相应的队列当中。序列头个元素为278，其最低位是8，278进入队列8（开头队列为空，则所进入元素将成为队列头）。对于109、63、930，则分别进入队列9、队列3、队列0。对于589，其必然进入队列9，由于队列9已有1个元素109，109也为当前的队尾，则589紧跟其后，成为新的队尾。对于剩余的元素都是同样的处理，则第1趟处理后，所有队列的情况为：队列0：930；队列1：空；队列2：空；队列3：063、083；队列4：184；队列5：505；队列6：空；队列7：空；队列8：278、008；队列9：109、589、269。我们从队列0开始一直到队列9，把前一个队列的队尾与后一个队列的队头链接起来，队列0的队头除外，其作为链接结果的头，队列9的队尾也除外，其作为链接结果的尾。从头到尾遍历这个链接结果，则得到第1趟处理后的序列：930、063、083、184、505、278、008、109、589、269。不难看出，该序列已经按元素最低位有序了。

　　然后，我们将清空全部队列，对第1趟处理后得到的序列，按照每个元素的中间位，完全效仿上文所介绍的步骤处理。则第2趟处理后的各队列如下：队列0：505、008、109；队列1：空；队列2：空；队列3：930；队列4：空；队列5：空；队列6：063、269；队列7：278；队列8：083、184、589；队列9：空。按同样方法把所有队列链接起来后遍历，得第2趟处理后的序列：505、008、109、930、063、269、278、083、184、589。这个序列显然按元素的中间位有序，读者或许在这里会提出个问题：第1趟处理后，序列按元素最低位有序，第2趟处理后，序列按元素中间位有序。这两者是相互独立没关系的吗？这两者究竟有何意义？细心观察可发现，第2趟处理的结果似乎破坏了第1趟处理的结果，让人觉得不知道是什么目的，比如说：第1趟处理后的505必然排在063之后，而第2趟处理必有505在063之前，这似乎让人觉得不可思议。然而，由于第2趟处理是在第1趟处理所得的序列上进行的，第1趟处理后，序列已经按元素的最低位有序。这点对于中间位相同的元素也不例外，那么显然，第2趟处理后的序列，尽管拥有不同中间位的元素将不再与第1趟处理后的顺序一致，但拥有相同中间位的元素将仍然保留第1趟处理后的顺序。比如505、008这两者，第1趟处理后，505必然在008前，由于两者中间位相同，第2趟处理后，505与008都在队列0，且505仍在008之前。由此，第2趟处理结果将有：序列按元素的中间位有序，与此同时，中间位相同的元素按最低位有序。

　　最后，我们再次清空全部队列，对第2趟排序后的序列，按每个元素的最高位，完全效仿上文所介绍的步骤同样处理。此刻将有：队列0：008、063、083；队列1：109、184；队列2：269、278；队列3：空；队列4：空；队列5：505、589；队列6：空；队列7：空；队列8：空；队列9：930。同样链接全部队列并遍历，得第3趟处理后序列：008、063、083、109、184、269、278、505、589、930。按照同样的分析，可知：第3趟排序后的序列按元素最高位有序，最高位相同的元素将会按中间位有序，中间位也相同者则会按最低位有序。这样的结果不正好是我们所期待的最终有序序列吗？所以说，至此，基数排序完毕，最后别忘了，可以把所有元素的前导0清除掉了。

　　从上面的介绍看来，基数排序明显是稳定排序。代码如下（注：代码中用到的队列，实际上可以是直接使用STL里头提供的队列，此处本人只是因为强迫症，而去实现了自己的队列版本而已，可忽略）：

{% highlight c %}
#include <cmath>

using namespace std;

struct ListNode
{
    int data;
    ListNode * next;
};
class Queue
{
    private:
        ListNode * head;
        ListNode * tail;
    public:
        Queue()
        {
            head=tail=0;
        }
        bool isEmpty()
        {
            return (head==0)&&(tail==0);
        }
        void enter(int element)
        {
            ListNode * listNode=new ListNode;
            listNode->data=element;
            listNode->next=0;
            if(isEmpty()) head=tail=listNode;
            else
            {
                    tail->next=listNode;
                    tail=listNode;
            }
        }
        int get()
        {
            return head->data;
        }
        void depart()
        {
            ListNode * temp=head;
            if(head==tail) head=tail=0;
            else head=head->next;
            delete temp;
        }
        ~Queue()
        {
            while(!isEmpty()) depart();
        }
};

int getMax(int list[],int length)
{
    int max=list[0];
    for(int i=1;i<length;++i)
    {
        if(list[i]>max) max=list[i];
    }
    return max;
}
int getNumOfDigits(int number)
{
    if(number==0) return 1;
    int numOfDigits=0;
    while(number!=0)
    {
        ++numOfDigits;
        number/=10;
    }
    return numOfDigits;
}

void radixSort(int list[],int length)
{
    int max=getMax(list,length);
    int numOfDigits=getNumOfDigits(max);
    for(int turn=1;turn<=numOfDigits;++turn)
    {
        Queue * queues=new Queue [10];
        for(int i=0;i<length;++i)
        {
            int digit=(list[i]/(int)pow(10,turn-1))%10;
            queues[digit].enter(list[i]);
        }
        int i=0;
        for(int j=0;j<10;++j)
        {
            while(!queues[j].isEmpty())
            {
                list[i++]=queues[j].get();
                queues[j].depart();
            }
        }
        delete [] queues;
    }
}
{% endhighlight %}

　　设序列元素个数为n，元素均为 r 进制数，且元素被统一定为d位。显然，基数排序的时间复杂度分析不存在最好最坏情况之分 。d决定了基数排序的趟数，每一趟处理中，需要扫描整个序列，以令元素按位进入相应队列，故这里会进行n次操作。然后，顺次地从头到尾遍历所有的队列，队列数由 r 决定，以更新原序列。这步需要进行n+r 次操作。所以每趟处理共需要进行2n+r 次操作，则基数排序的过程总共需要（2n+r）d 次处理。因此，基数排序的时间复杂度为O((n+r)d)。由于基数排序需要使用队列作为辅助存储空间，队列的数量、大小恰好与 r、n有关，上面提到过 r 即为队列数，而所有队列所用空间的大小主要由n线性决定。故基数排序的空间复杂度为O（n+r）。