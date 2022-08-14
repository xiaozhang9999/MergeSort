# MergeSort
实现测试归并排序及leetcode剑指Offer51.数组中的逆序对用归并排序算法解
# 归并排序法
## 自顶向下使用递归实现
对数组arr[]，最左边l，中间mid，最右边r；
MergeSort(arr,l,r),对arr[l,r]进行排序；
l>=r时return；
```c++
template <typename T>
void sort1(T arr[], int l, int r) {
	if (l >= r)
		return;
	int mid = l + (r-l) / 2;
	sort1(arr, l, mid);
	sort1(arr, mid + 1, r);
	merge(arr, l, mid, r);
}
```
### 归并过程
将两个有序数组A：arr[l,mid]、B:arr[mid+1,r]合并为一个有序数组arr[l,r]。
需要新声明一个数组temp[]拷贝arr数组实现,此时temp从0开始，arr从l开始，因此有l的偏移。
```c++
template <typename T>
void merge(T arr[], int l, int mid, int r)
{
	//T *temp=new T[length];//错误这种不能实现复制
	//memcpy(temp, arr, sizeof(arr));
	//使用vector动态数组
	vector<T>temp;
	temp.reserve(r - l + 1);//预分配空间
	temp.assign(&arr[l], &arr[r+1]);//拷贝赋值
	int i = l;
	int j = mid + 1;
	for (int k = l; k <= r; k++) {
		if (i > mid) {
			arr[k] = temp[j-l]; j++;
		}
		else if (j > r) {
			arr[k] = temp[i - l]; i++;
		}
		else if (temp[i-l] <= temp[j-l]) {
			arr[k] = temp[i-l]; i++;
		}
		else {
			arr[k] = temp[j-l]; j++;
		}
	}
}
```
### 复杂度分析
归并排序性能大于选择排序，插入排序；时间复杂度为O(nlogn)。
若左边有序数组全部小于右边有序数组，则不需要执行归并函数merge，可对此加if语句进行优化。
```c++
//对执行归并加条件，进行排序优化
template <typename T>
void sort2(T arr[], int l, int r) {
	if (l >= r)
		return;
	int mid = l + (r - l) / 2;
	sort2(arr, l, mid);
	sort2(arr, mid + 1, r);
	if (arr[mid] > arr[mid + 1]) {
	merge(arr, l, mid, r);
	}	
}
```
因为在归并排序中新创建了数组来实现归并，所以空间复杂度为O(n)。
优化后，对于有序数组归并排序时间复杂度为O(n)。
### 使用插入排序法优化归并排序
当数据规模较小时，使用插入排序法更好一些，所以在归并时可以加入限制条件使当r-l<=15时，使用插入排序。
```c++
//插入排序算法
template <typename T>
void InsertSort(T arr[], int l, int r)
{
	
	for (int i = l; i <= r; i++) {
		T temp = arr[i];
		int j;
		for (j = i; j > l&& temp < arr[j - 1]; j--) {
				arr[j] = arr[j - 1];
		}
		arr[j] = temp;
	}
}

//使用插入排序优化
template <typename T>
void sort3(T arr[], int l, int r) {
	/*if (l >= r)
		return;*/
	//当数据规模小时使用插入排序
	if ((r - l) <= 15) {
		InsertSort(arr, l, r);
		return;
	}
	int mid = l + (r - l) / 2;
	sort3(arr, l, mid);
	sort3(arr, mid + 1, r);
	if (arr[mid] > arr[mid + 1]) {
		merge(arr, l, mid, r);
	}
}
```
此优化对于现在高级语言使用意义可能不大。
### 内存操作优化
在merge()方法中，重复创建临时数组，会耗费一定时间，可以提前声明一个temp[]数组拷贝arr中内容，再以参数形式传入merge中，再在merge中复制需要归并的对应范围arr数组的内容。
```c++
template <typename T>
void merge2(T arr[], int l, int mid, int r,T temp[])
{
	//更新temp[l.r]=arr[l,r]
	for (int t = l; t <= r; t++) {
		temp[t] = arr[t];
	}
	int i = l;
	int j = mid + 1;
	for (int k = l; k <= r; k++) {
		if (i > mid) {
			arr[k] = temp[j]; j++;
		}
		else if (j > r) {
			arr[k] = temp[i]; i++;
		}
		else if (temp[i] <= temp[j]) {
			arr[k] = temp[i]; i++;
		}
		else {
			arr[k] = temp[j]; j++;
		}
	}
}
```
## 自底向上实现归并排序
sz为合并元素个数，初始sz=1,之后sz=sz+sz;合并arr[i,i+sz-1]和arr[i+sz,min(i+sz+sz-1,n-1)];最后元素个数不一定时sz整数倍。
```c++
template <typename T>
void sortBU(T arr[],int n) {
	T* temp = new T[n];
	for (int sz = 1; sz <= n; sz += sz) {
		//合并arr[i,i+sz-1]和arr[i+sz,min(i+sz+sz-1,n-1)]  n为数组大小
		for (int i = 0; i < n - sz; i += sz + sz) {
			int l = i, mid = i + sz - 1;
			int r = min(i + sz + sz - 1, n - 1);
			merge2(arr, l, mid, r,temp);
		}
	}
}
```
主函数测试
```c++
int main()
{
	const int n = 50000;//声明数组时[]内数组大小必须为常量，可以设为const，这时可以用int arr[n]声明数组，但是保存在堆栈中数组容量有限
	//int n = 50000;
	//int* arr = new int[n]; //或者也可以声明为动态数组,可以声明更大的数组,因为复制啥的就结果不对
	int arr[n];
	generateRandomArray(arr, n, n);//生成容量为n的0-n测试数组
	int arr1[n];
	memcpy(arr1, arr, sizeof(arr));//复制相同数组
	/*for (int i = 0; i < n;i++) {
		cout << arr[i] << " ";
	}
	cout << endl;
	for (int j = 0; j < n;j++) {
		cout << arr1[j] << " ";
	}
	cout << endl;*/
//	test("MergeSort1",arr, n);
//	test("MergeSort2", arr1, n);
//	test("MergeSort3", arr, n);
	test("MergeSort4", arr1, n);
	test("MergeSortBU", arr, n);
}
```
# 数组的逆序对数
对于leetcode 中剑指Offer51.题 数组中的逆序对
可以用简单的两层循环遍历，这样时间复杂度为O($n^2$)，超出了时间限制。可以用归并排序算法解决，在排序中即可得到逆序对数。
在归并函数中，将前后两部分有序序列合并为一个完全有序的序列，在这个过程中每次只有满足arr[i]>arr[j]时，此时角标i到角标mid的元素均大于角标为j的元素因此他们与后者均为逆序对,这样将提前声明好的计数器ans加上这些元素的个数，直到完成整个排序过程，计数器已经累计了所有的逆序对。因此只需要在merge()函数中加入一句 ans+=mid-i+1。
此为在leetcode 中的解答；
```c++
class  Solution {
public:
int ans=0;
int  reversePairs(vector<int>&  nums) {
ans=0;
vector<int>temp(nums.begin(),nums.begin());
sort(nums, 0, nums.size() - 1,temp);
return ans;
}
void  merge(vector<int>&arr, int  l, int  mid, int  r,vector<int>&temp)
{
temp.assign(arr.begin(),arr.end());
int i = l;
int j = mid + 1;
for (int k = l; k <= r; k++) {
if (i > mid) {
arr[k] = temp[j]; j++;
}
else  if (j > r) {
arr[k] = temp[i]; i++;
}
else  if (temp[i] <= temp[j]) {
arr[k] = temp[i]; i++;
}
else {
ans+=mid-i+1;
arr[k] = temp[j]; j++;
}
}
}
void  sort(vector<int>&arr, int  l, int  r,vector<int>&temp) {
if (l >= r)
return;
int mid = l + (r - l) / 2;
sort(arr, l, mid,temp);
sort(arr, mid + 1, r,temp);
if (arr[mid] > arr[mid + 1]) {
merge(arr, l, mid, r,temp);
}
}
};
```
# 小结
在本周的学习中更详细的掌握了归并排序算法，进一步加深了对于递归方法的理解。另外通过对算法的一步步优化掌握更多的编程的理解，在测试算法时遇到了关于声明数组的问题，在C++中静态数组不能创建得太大，而用new创建动态数组时会遇到各种内存方面问题联系到在内存复制数组一时难以顺利实现排序，最后使用const常量来声明了静态数组。
