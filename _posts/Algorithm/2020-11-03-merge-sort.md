

### 归并排序

时间复杂度 O(n log n)

空间复杂度 O(n) 因为需要单独开辟help数组空间来记录每次的merge

比较行为记录下来，左组和右组比较，之后保留结果

其他的O(n^2) 的排序，浪费了比较行为



```
/**
 * @ClassName MergeSort
 * @Description 归并排序
 * @Author kennedyhan
 * @Date 2020/11/15 0015 20:26
 * @Version 1.0
 **/
public class MergeSort {

    public static void main(String[] args) {
        int[] arrs = {5, 6, 1, 2, 9, 3, 3, 0, 7, 7};
        mergeSort(arrs, 0, arrs.length - 1);
        System.out.println(Arrays.toString(arrs));
    }

    /**
     * 采用分而治之思想
     * 左组有序，右组有序，再 merge
     * @param arr
     * @param L
     * @param R
     */
    public static void mergeSort(int[] arr, int L, int R) {
        if ( L == R ) { // 递归函数的 base case 当 L==R 终止递归（弹出系统栈）
            return;
        }

        // 取中点，和  (R + L) / 2 效果一样
        int mid = L + ((R - L) >> 1) ;
        mergeSort(arr, L, mid);
        mergeSort(arr, mid + 1, R);
        merge(arr, L, mid, R);
    }

    private static void merge(int[] arr, int L, int M, int R) {
        int[] help = new int[R - L + 1];    //help数组用来记录merge后的结果
        int i = 0;  //从0位置开始使用help数组
        int p1 = L; // p1 从L出发 到 M
        int p2 = M + 1; // p2从 M+1出发 到 R
        while (p1 <= M && p2 <= R) {
            // arr[p1] 和 arr[p2] 一样大时，先拷贝p2
            help[i++] = arr[p1] <= arr[p2] ? arr[p1++] : arr[p2++];
        }

        // 要么 p1越界，要么 p2越界，下面 2个while 只能触发其中 1个
        while (p1 <= M) {
            help[i++] = arr[p1++];
        }
        while (p2 <= R) {
            help[i++] = arr[p2++];
        }

        // 将help数组merge的结果，写回原数组arr中
        for (i = 0; i < help.length; i++) {
            arr[L + i] = help[i];
        }
    }
}

```





### 应用：在一个数组中，一个数左边比他小的数的总和，叫数的小和，所有的小和累加起来，叫数组小和。求数组小和

小和问题

就是求 每一个数，右边有多少个数比他大

利用右侧有序，把个数炸出来

merge上做改动

变化：

相比mergeSort，改变了返回值为int，merge返回值int

merge新增res保存累加结果，并返回

```
/**
 * @ClassName MergeSortSum
 * @Description 小和问题
 * 在一个数组中，一个数左边比他小的数的总和，叫数的小和，所有的小和累加起来，叫数组小和。求数组小和
 * @Author kennedyhan
 * @Date 2020/11/28 0028 18:13
 * @Version 1.0
 **/
public class MergeSortSum {

    public static void main(String[] args) {
        int[] arrs = {1, 3, 3, 2, 4};
        // 1 右边有4个数比1大，1*4
        // 3 右边有1个数比3大，3*1
        // 3 右边有1个数比3大，3*1
        // 2 右边有1个数比2大，2*1
        // 1*4 + 3*1 + 3*1 + 2*1 = 4+3+3+2 = 12
        int res = mergeSortSum(arrs, 0, arrs.length - 1);
        System.out.println(res);
    }

    /**
     * 采用分而治之思想
     * 左组有序，右组有序，再 merge
     * @param arr
     * @param L
     * @param R
     */
    public static int mergeSortSum(int[] arr, int L, int R) {
        if ( L == R ) { // 递归函数的 base case 当 L==R 终止递归（弹出系统栈）
            return 0;
        }

        // 取中点，和  (R + L) / 2 效果一样
        int mid = L + ((R - L) >> 1) ;
        return mergeSortSum(arr, L, mid)
                + mergeSortSum(arr, mid + 1, R)
                + merge(arr, L, mid, R);
    }

    private static int merge(int[] arr, int L, int M, int R) {
        int[] help = new int[R - L + 1];    //help数组用来记录merge后的结果
        int i = 0;  //从0位置开始使用help数组
        int p1 = L; // p1 从L出发 到 M
        int p2 = M + 1; // p2从 M+1出发 到 R
        int res = 0;	//记录本次merge的小和结果并返回
        while (p1 <= M && p2 <= R) {
        	//  左组比右组的数小时，产生小和，因为右组有序，此时右组有 R - p2 + 1 个数比左组的当前数大
            res += arr[p1] < arr[p2] ? (R - p2 + 1) * arr[p1] : 0;
            // arr[p1] 和 arr[p2] 一样大时，先拷贝p2
            help[i++] = arr[p1] < arr[p2] ? arr[p1++] : arr[p2++];
        }

        // 要么 p1越界，要么 p2越界，下面 2个while 只能触发其中 1个
        while (p1 <= M) {
            help[i++] = arr[p1++];
        }
        while (p2 <= R) {
            help[i++] = arr[p2++];
        }

        // 将help数组merge的结果，写回原数组arr中
        for (i = 0; i < help.length; i++) {
            arr[L + i] = help[i];
        }

        return res;
    }
}

```





### 应用：在一个数组中，求所有的逆序对

举例：数组 [3 1 7 0 2]

(3, 1) (3,0) (3,2)

(1,0)

(7,0)(7,2)

一共6个

`就是求 每一个数，右边有多少个数比他小`，和上一题正相反

p1 和 p2 从右往左遍历，因为数组从小到大有序，很容易找到右组有几个数比左组小

```
/**
 * @ClassName MergeSortSum
 * @Description 逆序对
 * 在一个数组中，求所有的逆序对
 * @Author kennedyhan
 * @Date 2020/11/28 0028 18:13
 * @Version 1.0
 **/
public class MergeSortReversePair {

    public static void main(String[] args) {
        int[] arrs = {3, 1, 7, 0, 2};
        // 3,1   3,0    3,2
        // 1,0
        // 7,0   7,2
        // 6个
        int res = mergeSortReversePair(arrs, 0, arrs.length - 1);
        System.out.println(res);
    }

    /**
     * 采用分而治之思想
     * 左组有序，右组有序，再 merge
     * @param arr
     * @param L
     * @param R
     */
    public static int mergeSortReversePair(int[] arr, int L, int R) {
        if ( L == R ) { // 递归函数的 base case 当 L==R 终止递归（弹出系统栈）
            return 0;
        }

        // 取中点，和  (R + L) / 2 效果一样
        int mid = L + ((R - L) >> 1) ;
        return mergeSortReversePair(arr, L, mid)
                + mergeSortReversePair(arr, mid + 1, R)
                + merge(arr, L, mid, R);
    }

    private static int merge(int[] arr, int L, int M, int R) {
        int[] help = new int[R - L + 1];    //help数组用来记录merge后的结果
        int i = help.length - 1;    // 从help数组末尾开始
        int p1 = M; // p1从M出发 走向 L
        int p2 = R; // p2从R出发 走向 M+1
        int res = 0;    //记录本次merge的结果
        while (p1 >= L && p2 > M) {
            //举例 左组 1 2 3 6 右组 2 4 4 6
            // p1=6 p2=6 先拷贝右组的，不产生逆序对
            // p1=6 p2=4 产生逆序对，所有右组都比左组p1小，记录3个，即 p2 - (M+1) + 1，之后拷贝左组，p1=3位置
            res += arr[p1] > arr[p2] ? (p2 - M) : 0;
            help[i--] = arr[p1] > arr[p2] ? arr[p1--] : arr[p2--];
        }

        // 要么 p1越界，要么 p2越界，下面 2个while 只能触发其中 1个
        while (p1 >= L) {
            help[i--] = arr[p1--];
        }
        while (p2 > M) {
            help[i--] = arr[p2--];
        }

        // 将help数组merge的结果，写回原数组arr中
        for (i = 0; i < help.length; i++) {
            arr[L + i] = help[i];
        }

        return res;
    }
}

```



### 应用: 一个数组中，每个数，右边有多少个数，乘2都没有当前的数大

举例：

{6,7,1,2,3}

从6开始往右看， 1*2 = 2 和 2 * 2 = 4 都没有6大，记2个

从7向右看，同理得到1、2、3分别乘2没有7大，记3个

1右边没有数乘2比1大，记0个

2和3同理，记0个

总个数=2+3=5个

