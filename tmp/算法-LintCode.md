## 1. A + B 问题
>   主要利用异或运算来完成。异或运算有一个别名叫做：不进位加法。  
>   那么a^b就是a和b相加之后，该进位的地方不进位的结果,然后下面考虑哪些地方要进位,自然是a和b里都是1的地方。    
>   a&b就是a和b里都是1的那些位置，a&b<<1 就是进位之后的结果。所以：a + b =(a ^ b) + (a & b << 1)。     
>   令a' = a ^ b, b' = (a & b) << 1。可以知道，这个过程是在模拟加法的运算过程，进位不可能一直持续，所以b最终会变为0。因此重复做上述操作就可以求得a + b的值。

	public int aplusb(int a, int b) {
	    while (b != 0) {
	        int _a = a ^ b;
	        int _b = (a & b) << 1;
	        a = _a;
	        b = _b;
	    }
	    return a;
	}

## 2. 尾部的零

	public long trailingZeros(long n) {
	    long sum = 0;
	    while (n / 5 != 0) {
	        n = n / 5;
	        sum += n;
	    }
	    return sum;
	}
## 3. 统计数字 PASS
## 4. 丑数 II

	public int nthUglyNumber(int n) {
	    int[] arr = new int[n];
	    arr[0] = 1;
	    int count_2 = 0;
	    int count_3 = 0;
	    int count_5 = 0;
	    for (int i = 1; i < n; i++) {
	        arr[i] = Math.min(Math.min(arr[count_2] * 2, arr[count_3] * 3), arr[count_5] * 5);
	        if (arr[i] / arr[count_2] == 2)
	            count_2++;
	        if (arr[i] / arr[count_3] == 3)
	            count_3++;
	        if (arr[i] / arr[count_5] == 5)
	            count_5++;
	    }
	    return arr[n  1];
	}

## 5. 第k大元素

```java
//方法一：依旧借助快排的思想，只不过每次只选择一边的子序列进行划分。每次选定基准之后，比基准值大的都在左边子序列，小的在右边子序列，所以只要将k与当前基准所在下标进行比较（基准下标index即为第index大的数），确认下次对哪半边子序列进行操作。
private int quicksort(int[] nums, int left, int right, int k) {
    int pivot = nums[left];
    int i = left, j = right;
    while (i <= j) {
        while (i <= j && nums[i] > pivot) {
            i++;
        }
        while (i <= j && nums[j] < pivot) {
            j--;
        }
        if (i <= j) {
            int tmp = nums[i];
            nums[i] = nums[j];
            nums[j] = tmp;
            i++;
            j--;
        }
    }
    if (left + k  1 <= j) {
        return quicksort(nums, left, j, k);
    }
    if (left + k  1 >= i) {
        return quicksort(nums, i, right, k  (i - left));
    }
    return nums[j + 1];
}
public int kthLargestElement(int k, int[] nums) {
    return quicksort(nums, 0, nums.length-1, k);
}
//方法二
class KthLargest {
    final PriorityQueue<Integer> q ;
    final int k;
    public KthLargest(int k, int[] nums) {
        this.k = k;
        q = new PriorityQueue<Integer>(k);
        for(int i: nums) {
            add(i);
        }
    }

    public int add(int val) {
        if(q.size() < k) {
            q.offer(val);

        }
        else if(q.peek() < val) {
            q.poll();
            q.offer(val);
        }
        return q.peek();
    }
}
```

## 6. 合并排序数组 II

```java
public int[] mergeSortedArray(int[] A, int[] B) {
    int arrLen = A.length + B.length;
    int[] arr = new int[arrLen];
    int i = 0;
    int j = 0;
    int k = 0;
    while (k < arrLen) {
        if (i >= A.length) {
            arr[k] = B[j];
            j++;
        } else if (j >= B.length) {
            arr[k] = A[i];
            i++;
        } else {
            if (A[i] > B[j]) {
                arr[k] = B[j];
                j++;
            } else {
                arr[k] = A[i];
                i++;
            }
        }
        k++;
   }
    return arr;
}
```

## 8. 旋转字符串
```java
//(X'Y')'=YX
public void rotateString(char[] str, int offset) {
    if (str == null || str.length == 0) {
        return;
    }
    offset = offset % str.length;
    reverse(str, 0, str.length - offset - 1);
    reverse(str, str.length - offset, str.length - 1);
    reverse(str, 0, str.length - 1);
}
private void reverse(char[] str, int start, int end) {
    for (int i = start, j = end; i < j; i++, j--) {
        char temp = str[i];
        str[i] = str[j];
        str[j] = temp;
    }
}
```
## 9. PASS
## 10. PASS


## 13. Implement strStr()

	public int strStr(String source, String target) {
	    if (source == null || target == null) {
	        return 1;
	    }
	    for (int i = 0; i < source.length()  target.length() + 1; i++) {
	        int j = 0;
	        while (j < target.length()) {
	            if (source.charAt(i + j) != target.charAt(j)) {
	                break;
	            }
	            j++;
	        }
	        if (j == target.length()) {
	            return i;
	        }
	    }
	    return 1;
	}

## 14.二分查找  PASS

## 15.全排列
	方法一：递归
	public List<List<Integer>> permute(int[] nums) {
		List<List<Integer>> results = new ArrayList<>();
		if (nums == null) {
			return results;
		}
		boolean[] visited = new boolean[nums.length];
		dfs(nums, visited, new ArrayList<Integer>(), results);
		return results;
	}
	
	private void dfs(int[] nums, boolean[] visited, List<Integer> permutation, List<List<Integer>> results) {
		if (nums.length == permutation.size()) {
			results.add(new ArrayList<Integer>(permutation));
			return;
		}
		for (int i = 0; i < nums.length; i++) {
			if (visited[i]) {
				continue;
			}
			permutation.add(nums[i]);
			visited[i] = true;
			dfs(nums, visited, permutation, results);
			visited[i] = false;
			permutation.remove(permutation.size() - 1);
		}
	}
	方法二：非递归，可用52题中的字典序法，循环求下一个排列
## 16. 带重复元素的排列
	public List<List<Integer>> permuteUnique(int[] nums) {
		List<List<Integer>> results = new ArrayList<>();
		if (nums == null) {
			return results;
		}
		Arrays.sort(nums);
		boolean[] visited = new boolean[nums.length];
		dfs(nums, visited, new ArrayList<Integer>(), results);
	
		return results;
	}
	
	private void dfs(int[] nums, boolean[] visited, List<Integer> permutation, List<List<Integer>> results) {
	
		if (nums.length == permutation.size()) {
			results.add(new ArrayList<Integer>(permutation));
			return;
		}
		for (int i = 0; i < nums.length; i++) {
			if (visited[i]) {
				continue;
			} else if (i != 0 && nums[i] == nums[i - 1] && !visited[i - 1]) {// 回溯的时候
				continue;
			}
			permutation.add(nums[i]);
			visited[i] = true;
			dfs(nums, visited, permutation, results);
			visited[i] = false;
			permutation.remove(permutation.size() - 1);
		}
	}
## 17.子集

	方法一：
	 public List<List<Integer>> subsets(int[] nums) {
	    List<List<Integer>> results = new LinkedList<>();
	    if (nums == null) {
	        return results; 
	    }
	    Arrays.sort(nums);
	    Queue<List<Integer>> queue = new LinkedList<>();
	    queue.offer(new LinkedList<Integer>());
	    while (!queue.isEmpty()) {
	        List<Integer> subset = queue.poll();
	        results.add(subset);    
	        for (int i = 0; i < nums.length; i++) {
	            if (subset.size() == 0 || subset.get(subset.size() - 1) < nums[i]) {
	                List<Integer> nextSubset = new LinkedList<Integer>(subset);
	                nextSubset.add(nums[i]);
	                queue.offer(nextSubset);
	            }
	        }
	    }
	    return results;
	}
	方法二：
	public List<List<Integer>> subsets(int[] nums) {
		List<List<Integer>> results = new LinkedList<>();
		if (nums == null) {
			return results;
		}
		Arrays.sort(nums);
		int setNum = 1 << nums.length;// 一共有2^n种可能
		for (int i = 0; i < setNum; i++) {
			List<Integer> item = new ArrayList();
			for (int j = 0; j < nums.length; j++) {
				if ((i & (1 << j)) != 0) {// 有没有第j个数
					item.add(nums[j]);
				}
			}
			results.add(item);
		}
		return results;
	}
	方法三：
	public List<List<Integer>> subsets(int[] nums) {
		List<List<Integer>> results = new LinkedList<>();
		if (nums == null) {
			return results;
		}
		Arrays.sort(nums);
		List<Integer> item = new ArrayList();
		helper(nums, 0, item, results);
		return results;
	}
	public void helper(int[] nums, int startIndex, List<Integer> subset, List<List<Integer>> results) {
		results.add(new ArrayList<Integer>(subset));
		for (int i = startIndex; i < nums.length; i++) {
			subset.add(nums[i]);
			helper(nums, i + 1, subset, results);
			subset.remove(subset.size() - 1);
		}
	}
## 18.Subsets II
	public List<List<Integer>> subsetsWithDup(int[] nums) {
		List<List<Integer>> results = new LinkedList<>();
		if (nums == null) {
			return results;
		}
		Arrays.sort(nums);
		List<Integer> item = new ArrayList();
		helper(nums, 0, item, results);
		return results;
	}
	public void helper(int[] nums, int startIndex, List<Integer> subset, List<List<Integer>> results) {
		results.add(new ArrayList<Integer>(subset));
		for (int i = startIndex; i < nums.length; i++) {
			if (i != startIndex && nums[i] == nums[i - 1]) {
				continue;
			}
			subset.add(nums[i]);
			helper(nums, i + 1, subset, results);
			subset.remove(subset.size() - 1);
		}
	}
## 19.20.21.22.23.PASS
## 24.LFU缓存
## 25.26.27.PASS
## 28.
## 29.

## 30. 插入区间

```java
public List<Interval> insert(List<Interval> intervals, Interval newInterval) {
    if (newInterval == null || intervals == null) {
        return intervals;
    }
    List<Interval> results = new ArrayList<Interval>();
    int insertPos = 0;
    for (Interval interval : intervals) {
        //和newInterval没有交集的newInterval前面的区间
        if (interval.end < newInterval.start) {
            results.add(interval);
            insertPos++;
        }
        //和newInterval没有交集的newInterval后面的区间
        else if (interval.start > newInterval.end) {
            results.add(interval);
        } else {
            newInterval.start = Math.min(interval.start, newInterval.start);
            newInterval.end = Math.max(interval.end, newInterval.end);
        }
    }
    results.add(insertPos, newInterval);
    return results;
}
```

## 31. 数组划分

```java
public int partitionArray(int[] nums, int k) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    boolean flag = false;
    int i = 0;
    int j = nums.length - 1;
    while (i < j) {
        // 从右向左找，<k的
        while (i < j && nums[j] >= k) {
            j;
        }
        // 从左向右找，>=k的
        while (i < j && nums[i] < k) {
            i++;
        }
        if (i < j) {
            int t = nums[i];
            nums[i] = nums[j];
            nums[j] = t;
            flag = true;
        }
    }
    if (i == nums.length){
        return nums.length -1;
    }
    if (j == 0&&!flag){
        return 0;
    }
    return i + 1;
}
```
## 32.最小子串覆盖
```java
public String minWindow(String source, String target) {
    String result = "";
    if ("".equals(source) || "".equals(target)) {
        return result;
    }
    int[] sHash = new int[128];
    int[] tHash = new int[128];
    for (char ch : target.toCharArray()) {
        tHash[ch]++;
    }
    int count = 0;
    int start = 0;
    int min = Integer.MAX_VALUE;
    int minS = -1;
    int end = source.length() - 1;
    for (int i = 0; i < source.length(); i++) {
        sHash[source.charAt(i)]++; // 计算每一个字符出现的频数
        if (sHash[source.charAt(i)] <= tHash[source.charAt(i)]) { // 判断该字符是否在target中出现过
            count++;
        }
        if (count == target.length()) { // 找到了一个子串
            // 为了让start指向第一个符合要求的字符。
            while (start < i && (sHash[source.charAt(start)] > tHash[source.charAt(start)])) {
                sHash[source.charAt(start)]--;// 还原无效的字符的频数
                start++;
            }
            if (i - start < min) {
                min = i - start;
                System.out.println(min);
                end = i;
                minS = start;
            }
            // 继续寻找下一个子串，start加1，复原freq在start位置的频数，count减1。
            // 因为答案的子串中的字母与target不需要具有相同的顺序
            sHash[source.charAt(start)]--;
            start++;
            count--;
        }
    }
    if (minS == -1) {
        return "";
    }
    return source.substring(minS, end + 1);
}
```
## 33.
## 34.


## 37.PASS
## 38.搜索二维矩阵 II
```java
public int searchMatrix(int[][] matrix, int target) {
    int row = matrix.length - 1;
    int column = 0;
    int ans = 0;
    while (row >= 0 && column < matrix[0].length) {
        if (target == matrix[row][column]) {
            ans++;
            row--;
            column++;
            continue;
        }
        if (target < matrix[row][column]) {
            row--;
        } else {
            column++;
        }
    }
    return ans;
}
```



## 41. 最大子数组

```java
public int maxSubArray(int[] A) {
    if (A == null || A.length == 0){
        return 0;
    }
    int max = Integer.MIN_VALUE;
    int sum = 0;
    for (int i = 0; i < A.length; i++) {
        sum += A[i];//sum记录从A[0]到A[i]之间的数的和
        max = Math.max(max, sum);  //现在子数组的最大值
        sum = Math.max(sum, 0);//如果sum小于零，则sum重新置零（放弃之前的各数和）
    }
    return max;
}
```




## 42.最大子数组 II
	public int maxTwoSubArrays(List<Integer> nums) {
		if (nums == null || nums.size() == 0) {
			return 0;
		}
		int[] left = new int[nums.size()];// 记录从0到当前位置(i)下的最大子数组的和
		int[] right = new int[nums.size()];// 记录从i至size-1中最大子数组的和
		int lsum = 0;
		int lmax = Integer.MIN_VALUE;
		for (int i = 0; i < nums.size(); i++) {
			lsum += nums.get(i);
			lmax = Math.max(lmax, lsum);
			left[i] = lmax;
			lsum = Math.max(lsum, 0);
		}
		int rsum = 0;
		int rmax = Integer.MIN_VALUE;
		for (int j = nums.size() - 1; j > -1; j--) {
			rsum += nums.get(j);
			rmax = Math.max(rmax, rsum);
			right[j] = rmax;
			rsum = Math.max(rsum, 0);
		}
		int max = Integer.MIN_VALUE;
		for (int i = 0; i < nums.size() - 1; i++) {
			max = Math.max(max, left[i] + right[i + 1]);
		}
		return max;
	}
## 43. 最大子数组 III
## 44. 最小子数组
```java
public int minSubArray(List<Integer> nums) {
    if (nums == null || nums.size() == 0){
        return 0;
	}
    int min = Integer.MAX_VALUE;
    int sum = 0;
    for (int i = 0; i < nums.size(); i++) {
        sum += nums.get(i);
        min = Math.min(min, sum);
        sum = Math.min(sum, 0);
    }
    return min;
}
```
## 45. 最大子数组差
```java
public int maxDiffSubArrays(int[] nums) {
	if (nums == null || nums.length == 0) {
		return 0;
	}
	int[] lMax = new int[nums.length];
	int[] lMin = new int[nums.length];
	int lmax = Integer.MIN_VALUE;
	int lmin = Integer.MAX_VALUE;
	int lMaxSum = 0;
	int lMinSum = 0;
	for (int i = 0; i < nums.length; i++) {
		lMaxSum += nums[i];
		lmax = Math.max(lmax, lMaxSum);
		lMax[i] = lmax;
		lMaxSum = Math.max(lMaxSum, 0);

		lMinSum += nums[i];
		lmin = Math.min(lmin, lMinSum);
		lMin[i] = lmin;
		lMinSum = Math.min(lMinSum, 0);
	}
	int[] rMax = new int[nums.length];
	int[] rMin = new int[nums.length];
	int rmax = Integer.MIN_VALUE;
	int rmin = Integer.MAX_VALUE;
	int rMaxSum = 0;
	int rMinSum = 0;
	for (int j = nums.length - 1; j >= 0; j--) {
		rMaxSum += nums[j];
		rmax = Math.max(rMaxSum, rmax);
		rMax[j] = rmax;
		rMaxSum = Math.max(0, rMaxSum);

		rMinSum += nums[j];
		rmin = Math.min(rmin, rMinSum);
		rMin[j] = rmin;
		rMinSum = Math.min(rMinSum, 0);
	}
	int diff = 0;
	for (int i = 0; i < nums.length - 1; i++) {
		diff = Math.max(diff, Math.abs(lMax[i] - rMin[i + 1]));
		diff = Math.max(diff, Math.abs(lMin[i] - rMax[i + 1]));
	}
	return diff;
}
```

## 46. 主元素

```java
public int majorityNumber(List<Integer> nums) {
    int currentMajor = 0;
    int count = 0;
    for(Integer num : nums) {
        if(count == 0) {
            currentMajor = num;
        }
        if(num == currentMajor) {
            count++;
        } else {
            count--;
        }
    }
    return currentMajor;
}
```

## 47. 主元素Ⅱ

```java
public int majorityNumber(List<Integer> nums) {
    int majorityNumber1 = 0;
    int majorityNumber2 = 0;
    int count1 = 0;
    int count2 = 0;
    for (int i = 0; i < nums.size(); i++) {
        if (majorityNumber1 == nums.get(i)) {
            count1++;
        } else if (majorityNumber2 == nums.get(i)) {
            count2++;
        } else if (count1 == 0) {
            majorityNumber1 = nums.get(i);
            count1++;
        } else if (count2 == 0) {
            majorityNumber2 = nums.get(i);
            count2++;
        } else {
            count1--;
            count2--;
        }
    }
    count1 = count2 = 0;
    for (int i = 0; i < nums.size(); i++) {
        if (majorityNumber1 == nums.get(i)) {
            count1++;
        } else if (majorityNumber2 == nums.get(i)) {
            count2++;
        }
    }
    return count1 > count2 ? majorityNumber1 : majorityNumber2;
}
```

## 48.
## 49.字符大小写排序
```java
public void sortLetters(char[] chars) {
	int i = 0;
	int j = chars.length - 1;
	while (i <= j) {
		while (i <= j && Character.isLowerCase(chars[i])){
			i++;
		}
		while (i <= j && Character.isUpperCase(chars[j])){
			j--;
		}
		if (i <= j) {
			char tmp = chars[i];
			chars[i] = chars[j];
			chars[j] = tmp;
			i++;
			j--;
		}
	}
	return;
}
```
## 50.PASS
## 51.上一个排列
	//字典序法
	public void swap(List<Integer> nums, int i, int j) {
		int t1 = nums.get(i);
		int t2 = nums.get(j);
		nums.set(i, t2);
		nums.set(j, t1);
	}
	public void reverse(List<Integer> nums, int i, int j) {
		while (i < j) {
			swap(nums, i, j);
			i++;
			j--;
		}
	}
	public List<Integer> previousPermuation(List<Integer> nums) {
		if (Objects.isNull(nums) || nums.size() <= 1) {
			return nums;
		}
		// 从右至左找第一个降序的两个元素。A[i]>A[i+1]
		int i = nums.size() - 2;
		while (i >= 0 && nums.get(i) <= nums.get(i + 1)) {
			i--;
		}
		if (i < 0) {
			Collections.sort(nums);
			reverse(nums, 0, nums.size() - 1);
			return nums;
		}
		// 从右至i，找到第一个比A[i]小的元素A[j]，交换A[i]和A[j]的位置
		int j = nums.size() - 1;
		while (j > i && nums.get(j) >= nums.get(i)) {
			j--;
		}
		swap(nums, i, j);
		// 反转i+1到最后的元素
		reverse(nums, i + 1, nums.size() - 1);
		for (int num : nums)
			System.out.println(num);
		return nums;
	}
## 52.下一个排列
	//字典序法
	public void swap(int[] nums, int i, int j) {
		int temp = nums[i];
		nums[i] = nums[j];
		nums[j] = temp;
	}
	public void reverse(int[] nums, int i, int j) {
		while (i < j) {
			swap(nums, i, j);
			i++;
			j--;
		}
	}
	public int[] nextPermutation(int[] nums) {
		if (Objects.isNull(nums) || nums.length <= 1) {
			return nums;
		}
		// 从右至左找第一个升序的两个元素。A[i]<A[i+1]
		int i = nums.length - 2;
		while (i >= 0 && nums[i] >= nums[i + 1]) {
			i--;
		}
		if (i < 0) {
			Arrays.sort(nums);
			return nums;
		}
		// 从右至i，找到第一个比A[i]大的元素A[j]，交换A[i]和A[j]的位置
		int j = nums.length - 1;
		while (j > i && nums[j] <= nums[i]) {
			j--;
		}
		swap(nums, i, j);
		// 反转i+1到最后的元素
		reverse(nums, i + 1, nums.length - 1);
		return nums;
	}
## 53. 翻转字符串
```java
//使用JAVA API
public String reverseWords(String s) {
	    if (s.equals(""))
	        return s;
	    String regex = "\\s+";
	    String[] strArr = s.trim().split(regex);
	
	    String newStr = "";
	    for (int i = strArr.length  1; i > 1; i) {
	        newStr += strArr[i];
	        newStr += " ";
	    }
    return newStr.trim();
}
//不使用JAVA API
public String reverseWords(String s) {
    if (s.equals(""))
        return s;
    String newStr = "";
    int subEnd = 1;
    boolean flag = false;
    for (int i = s.length()  1; i > 1; i) {
        char c = s.charAt(i);
        if (c != ' ' && !flag) {
            subEnd = i;
            flag = true;
        }
        if (c == ' ' && flag) {
            for (int j = i + 1; j <= subEnd; j++) {
                newStr += s.charAt(j);
            }
            flag = false;
            newStr += " ";
        } else if (flag && i == 0) {
            for (int j = i; j <= subEnd; j++) {
                newStr += s.charAt(j);
            }
            flag = false;
            newStr += " ";
        }
    }
    return newStr.trim();
}
```
## 54.PASS
## 两数之和-输入已排序的数组

给定一个已按照**升序排列** 的有序数组，找到两个数使得它们相加之和等于目标数。

```java
public int[] twoSum(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1;
    while (nums[left] + nums[right] != target) {
        if (nums[left] + nums[right] > target) {//
            right--;
        }
        if (nums[left] + nums[right] < target) {
            left++; 
        }
    }
    int[] ans = new int[2];
    ans[0] = left + 1;
    ans[1] = right + 1;
    return ans;
}
```
## 56. 两数之和
```java
public int[] twoSum(int[] numbers, int target) {
    if (numbers == null || numbers.length == 0)
        return null;
    int[] index = new int[2];
    HashMap<Integer, Integer> map = new HashMap<>(numbers.length);
    for (int i = 0; i < numbers.length; i++) {
        int num = numbers[i];
        int diff = target  num;
        Integer val = map.get(diff);
        if (val != null) {
            index[0] = Math.min(i, val);
            index[1] = Math.max(i, val);
            return index;
        } else {
            map.put(num, i);
        }
    }
    return null;
}
```

## 两数之和 - BST版本

```java
public int[] twoSum(TreeNode root, int n) {
    return helper(root, root, n);
}
private TreeNode search(TreeNode root, int val) {
    if (root == null) return null;
    if (val == root.val) return root;
    if (val < root.val) return search(root.left, val);
    return search(root.right, val);
}
private int[] helper(TreeNode root, TreeNode topRoot, int n) {
    if (root == null) return null;

    int ans[];
    TreeNode another = search(topRoot, n - root.val);
    if (another != null && another != root) {
        ans = new int[2];
        ans[0] = root.val;
        ans[1] = n - root.val;
        return ans;
    }
    ans = helper(root.left, topRoot, n);
    if (ans != null) {
        return ans;
    }
    return helper(root.right, topRoot, n);
}
```



## 三数之和

```java
public List<List<Integer>> threeSum(int[] numbers) {
    List<List<Integer>> result = new ArrayList<List<Integer>>();
    if (numbers == null || numbers.length < 3){
        return result;
    }
    Arrays.sort(numbers);
    for (int i = 0; i < numbers.length; i++) {
        int left = i + 1;
        int right = numbers.length  1;
        while (left < right) {
            int sum = numbers[i] + numbers[left] + numbers[right];
            ArrayList<Integer> path = new ArrayList<Integer>();
            if (sum == 0) {
                path.add(numbers[i]);
                path.add(numbers[left]);
                path.add(numbers[right]);
                if (result.contains(path) == false)
                    result.add(path);
                left++;
                right;
            } else if (sum > 0) {
                right;
            } else {
                left++;
            }
        }
    }
    return result;
}
```

## 58.

## 59. 最接近的三数之和

	public int threeSumClosest(int[] numbers, int target) {
	    if (numbers == null || numbers.length < 3) {
	        return 0;
	    }
	    Arrays.sort(numbers);
	    int SumClosest = Integer.MAX_VALUE;
	    for (int i = 0; i < numbers.length; i++) {
	        int start = i + 1, end = numbers.length  1;
	        while (start < end) {
	            int sum = numbers[i] + numbers[start] + numbers[end];
	            if (Math.abs(target  sum) < Math.abs(target  SumClosest)) {
	                SumClosest = sum;
	            }
	            if (sum < target) {
	                start++;
	            } else {
	                end;
	            }
	        }
	    }
	    return SumClosest;
	}
## 60. 搜索插入位置
	public int searchInsert(int[] A, int target) {
	    if (A == null || A.length == 0) {
	        return 0;
	    }
	    int start = 0;
	    int end = A.length -1;
	    while (start < end) {
	        int mid = (start + end) / 2;
	        if (A[mid] == target) {
	            return mid;
	        } else if (A[mid] < target) {
	            start = mid + 1;
	        } else {
	            end = mid  1;
	        }
	    }
	    if (A[start] >= target) {
	        return start;
	    } else if (A[end] >= target) {
	        return end;
	    } else {
	        return end + 1;
	    }
	}
## 61. 搜索区间
	public int[] searchRange(int[] A, int target) {
	    int[] ret = { 1, 1 };
	    if (A.length == 0) {
	        return ret;
	    }
	    int i = 0;
	    int j = A.length  1;
	    while (i <= j) {
	        int mid = (i + j) / 2;
	        int num = A[mid];
	        if (target > num) {
	            i = mid + 1;
	        } else if (target < num) {
	            j = mid  1;
	        } else if (target == num) {
	            i = mid;
	            j = mid;
	            while (i > 1 && A[i] == target) {
	                ret[0] = i;
	                i;
	            }
	            while (j < A.length && A[j] == target) {
	                ret[1] = j;
	                j++;
	            }
	            break;
	        }
	    }
	    return ret;
	}
## 62.搜索旋转排序数组
```java
public int search(int[] A, int target) {
    if (A == null || A.length == 0) {
        return 1;
    }

    int start = 0;
    int end = A.length-1;
    int mid;

    while (start + 1 < end) {
        mid = start + (end - start) / 2;
        if (A[mid] == target) {
            return mid;
        }
        if (A[start] < A[mid]) {
            if (A[start] <= target && target <= A[mid]) {
                end = mid;
            } else {
                start = mid;
            }
        } else {
            if (A[mid] <= target && target <= A[end]) {
                start = mid;
            } else {
                end = mid;
            }
        }
    } 
    if (A[start] == target) {
        return start;
    }
    if (A[end] == target) {
        return end;
    }
    return 1;
}
```
## 63. 搜索旋转排序数组 II PASS
## 64. PASS



## 72. 中序遍历和后序遍历树构造二叉树
```java
private int findPosition(int[] arr, int start, int end, int key) {
    int i;
    for (i = start; i <= end; i++) {
        if (arr[i] == key) {
            return i;
        }
    }
    return -1;
}

// 后序找根，根节点就是后序遍历中的最后一位，中序找子树
private TreeNode myBuildTree(int[] inorder, int instart, int inend, int[] postorder, int poststart, int postend) {
    if (instart > inend) {
        return null;
    }

    TreeNode root = new TreeNode(postorder[postend]);
    int position = findPosition(inorder, instart, inend, postorder[postend]);

    root.left = myBuildTree(inorder, instart, position - 1, postorder, poststart, poststart + position - instart - 1);
    root.right = myBuildTree(inorder, position + 1, inend, postorder, poststart + position - instart, postend - 1);
    return root;
}

public TreeNode buildTree(int[] inorder, int[] postorder) {
    if (inorder.length != postorder.length) {
        return null;
    }
    return myBuildTree(inorder, 0, inorder.length - 1, postorder, 0, postorder.length - 1);
}
```
## 73. 前序遍历和中序遍历树构造二叉树
```java
private int findPosition(int[] arr, int start, int end, int key) {
    int i;
    for (i = start; i <= end; i++) {
        if (arr[i] == key) {
            return i;
        }
    }
    return -1;
}

// 先序找根，中序找子树
private TreeNode myBuildTree(int[] inorder, int instart, int inend, int[] preorder, int prestart, int preend) {
    if (instart > inend) {
        return null;
    }

    TreeNode root = new TreeNode(preorder[prestart]);
    int position = findPosition(inorder, instart, inend, preorder[prestart]);

    root.left = myBuildTree(inorder, instart, position - 1, preorder, prestart + 1, prestart + position - instart);
    root.right = myBuildTree(inorder, position + 1, inend, preorder, position - inend + preend + 1, preend);
    return root;
}

public TreeNode buildTree(int[] preorder, int[] inorder) {
    if (inorder.length != preorder.length) {
        return null;
    }
    return myBuildTree(inorder, 0, inorder.length - 1, preorder, 0, preorder.length - 1);
}
```

## 75. 寻找峰值

	public int findPeak(int[] A) {
	    int start = 1, end = A.length  2;  
	    while (start + 1 < end) {
	        int mid = (start + end) / 2;
	        if (A[mid] < A[mid  1]) {
	            end = mid;
	        } else if (A[mid] < A[mid + 1]) {
	            start = mid;
	        } else {
	            end = mid;
	        }
	    }
	    if (A[start] < A[end]) {
	        return end;
	    } else {
	        return start;
	    }
	}
## 76. 最长上升子序列
```java
public int longestIncreasingSubsequence(int[] nums) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    // dp[i]中存储以nums[i]为结尾的最长上升子序列的长度
    int[] dp = new int[nums.length];
    int max = 0;
    for (int i = 0; i < nums.length; i++) {
        dp[i] = 1;
        // 遍历i之前的数组，看Nums[i]能否加入到以nums[j]为结尾的最长上升子序列中
        for (int j = 0; j < i; j++) {
            // 如果nums[i] > nums[j]，nums[i]可以加入到以nums[j]为结尾的最长上升子序列中
            if (nums[j] < nums[i]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        max = Math.max(dp[i], max);
    }
    return max;
}
//求最长公共子序列
public int longestIncreasingSubsequence(int[] nums) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    int index = 0;
    int[] pre = new int[nums.length];// 存储前缀
    // dp[i]中存储以nums[i]为结尾的最长上升子序列的长度
    int[] dp = new int[nums.length];
    int max = 0;
    for (int i = 0; i < nums.length; i++) {
        dp[i] = 1;
        pre[i] = -1;
        // 遍历i之前的数组，看nums[i]能否加入到以nums[j]为结尾的最长上升子序列中
        for (int j = 0; j < i; j++) {
            // 如果nums[i] > nums[j]，nums[i]可以加入到以nums[j]为结尾的最长上升子序列中
            if (nums[j] < nums[i] && dp[i] < dp[j] + 1) {
                dp[i] = dp[j] + 1;
                pre[i] = j;
            }
        }
        if (dp[i] > max) {
            max = dp[i];
            index = i;
        }
    }
    List<Integer> list = new ArrayList();
    while (index >= 0) {
        list.add(nums[index]);
        index = pre[index];
    }
    Collections.reverse(list);
    return max;
}
```
## 77.最长公共子序列
```java
public int longestCommonSubsequence(String A, String B) {
    if(A.length==0||B.length==0){
        return 0;
    }
	int f[][] = new int[A.length() + 1][B.length() + 1];
	for (int i = 1; i <= A.length(); i++) {
		for (int j = 1; j <= B.length(); j++) {
			f[i][j] = Math.max(f[i - 1][j], f[i][j - 1]);
			if (A.charAt(i - 1) == B.charAt(j - 1)) {
				f[i][j] = f[i - 1][j - 1] + 1;
			}
		}
	}
	return f[A.length()][B.length()];
}
```
## 79.最长公共子串
```java
算法思路：
1、把两个字符串分别以行和列组成一个二维矩阵。
2、比较二维矩阵中每个点对应行列字符中否相等，相等的话值设置为1，否则设置为0。
3、通过查找出值为1的最长对角线就能找到最长公共子串。
public int longestCommonSubstring(String A, String B) {
	if (A == null && B != null || A != null && B == null) {
		return 0;
	} else if (A.equals(B)) {
		return A.length();
	}
	int m = A.length();
	int n = B.length();
	int max = 0;
	int[][] len = new int[m + 1][n + 1];
	for (int i = 0; i < m; i++) {
		for (int j = 0; j < n; j++) {
			if (A.charAt(i) == B.charAt(j)) {
				len[i + 1][j + 1] = len[i][j] + 1;
			} else {
				len[i + 1][j + 1] = 0;
			}
			max = Math.max(max, len[i + 1][j + 1]);
		}
	}

	return max;
}
```
## 81.数据流的中位数
```java
public int[] medianII(int[] nums) {
    if (nums == null || nums.length == 0) {
        return new int[0];
    }
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder()); // left
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    int[] result = new int[nums.length];
    for (int i = 0; i < nums.length; i++) {
        addNums(maxHeap, minHeap, nums[i]);
        result[i] = maxHeap.peek();
    }
    return result;
}

private void addNums(PriorityQueue<Integer> maxHeap, PriorityQueue<Integer> minHeap, int num) {
    maxHeap.offer(num); // 1.新加入的元素先入到大根堆，由大根堆筛选出堆中最大的元素
    minHeap.offer(maxHeap.poll()); // 2.筛选后的【大根堆中的最大元素】进入小根堆
    if (minHeap.size() - maxHeap.size() > 0) {
        System.out.println(minHeap.peek());
        maxHeap.offer(minHeap.poll());
    }
}
```
## 82. 落单的数
	方法一：
	public int singleNumber(int[] A) {
	    if (A == null || A.length == 0){
	        return 0;
		}
	    int xor = 0;
	    for (int i = 0; i < A.length; i++) {
	        xor ^= A[i];
	    }
	    return xor;
	}
	方法二：参见83题
## 83.落单的数 II
	思路:利用位运算，int有32位，用一个长度为32的数组bit记录A中每个数字的每一位中1出现的次数，如果这个数字出现3次，则与这个数字对应的每一位上的1也出现三次。最后将数组每一位均对3取余，最后得到的就是要求的数字。
	public int singleNumberII(int[] A) {
		if (A == null || A.length == 0) {
			return 0;
		}
		int val = 0;
		int[] bit = new int[32];
		for (int i = 0; i < 32; i++) {
			for (int j = 0; j < A.length; j++) {
				bit[i] += (A[j] >> i) & 1;//不能让1右移i位，只能是A[j]左移i位
			}
			bit[i] %= 3;
			val |= (bit[i] << i);
		}
		return val;
	}
## 84. 落单的数 III
```java
/*
思路:对于2*n+1个数字用异或就可以，参见博客LintCode-82.落单的数，而在此题将所有数异或之后得到的是两个落单的数的异或结果，没办法将结果拆分成两个落单的数。但因为两个落单数不同，所以肯定存在某个位k，使得两落单数在第k位上一个为0另一个为1（怎么找到这个k? 找异或结果中1出现的位置即可）。只需找到最小的这个k，然后将在k位上为0的所有数做异或得出其中一个落单的数，在k位为1的所有数也做另外的异或，得出另一个落单的数，这样最终可以得到两个落单的数。
*/
public List<Integer> singleNumberIII(int[] A) {
    ArrayList<Integer> result = new ArrayList<Integer>(2);
    if (A == null || A.length == 0) {
        return result;
    }
    int xor = 0;
    for (int i = 0; i < A.length; i++) {
        xor ^= A[i];
    }
    int lastBit = xor - (xor & (xor - 1));
    int v1 = 0;
    int v2 = 0;
    for (int i = 0; i < A.length; i++) {
        if ((lastBit & A[i]) == 0) {
            v1 ^= A[i];
        } else {
            v2 ^= A[i];
        }
    }
    result.add(v1);
    result.add(v2);
    return result;
}
```


<<<<<<< HEAD

=======
方法二：递归
public TreeNode insertNode(TreeNode root, TreeNode node) {
    if (root == null) {
        return node;
    }
    if (root.val > node.val) {
        root.left = insertNode(root.left, node);
    } else {
        root.right = insertNode(root.right, node);
    }
    return root;
}
```
## 88. Lowest Common Ancestor of a Binary Tree
​```java
//思路:
//1 如果A或B就在root上，那么root就是LCA。
//2 如果左子树和右子树分别都有LCA，那么root就是LCA。
//3 如果右子树没有LCA，左子树有，那么LCA在左子树。
//4 如果左子树没有LCA，右子树右，那么LCA在右子树。
//5 如果两边都没有，那么就没有。
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode A, TreeNode B) {
	if (root == null || root == A || root == B) {
		return root;
	}
	TreeNode left = lowestCommonAncestor(root.left, A, B);
	TreeNode right = lowestCommonAncestor(root.right, A, B);

	if (left != null && right != null) {
		return root;
	}
	if (left != null) {
		return left;
	}
	if (right != null) {
		return right;
	}
	return null;
}
```
>>>>>>> 修改
## 90. k数和 II
	public List<List<Integer>> kSumII(int[] A, int k, int target) {
		List<List<Integer>> result = new ArrayList<>();
		if (A == null || A.length == 0) {
			return result;
		}
		List<Integer> item = new ArrayList<Integer>();
		dfs(A, k, target, A.length - 1, result, item);
		return result;
	}
	
	private void dfs(int[] A, int k, int target, int n, List<List<Integer>> result, List<Integer> item) {
		if (target == 0 && k == 0) {
			result.add(new ArrayList(item));
			return;
		}
		if (k < 0 || target < 0) {
			return;
		}
		for (int i = n; i > -1; i--) {
			item.add(A[i]);
			target -= A[i];
			dfs(A, k - 1, target, i - 1, result, item);
			item.remove(item.size() - 1);
			target += A[i];
		}
	}

## 92. 背包问题

	public int backPack(int m, int[] A) {
	    int[] dp = new int[m + 1];
	    for (int i = 0; i < A.length; i++) {
	        for (int j = m; j > 0; j) {
	            if (j >= A[i]) {
	                dp[j] = Math.max(dp[j], dp[j  A[i]] + A[i]);
	            }
	        }
	    }
	    return dp[m];
	}



## 96. 链表划分
	//用两个链表表示，分别存储小于x的节点和大于等于x的节点



## 100. 删除排序数组中的重复数字

```java
public int removeDuplicates(int[] nums) {
    if (nums == null || nums.length == 0)
        return 0;
    int t = nums[0];
    int count = 0;

    for (int i = 1; i < nums.length  count; i++) {
        if (nums[i] == t) {
            for (int j = i; j < nums.length  1; j++) {
                nums[j] = nums[j + 1];
            }
            i;
            count++;
        } else {
            t = nums[i];
        }
    }
    return nums.length  count;
}
```

## 101. 删除排序数组中的重复数字 II
	public int removeDuplicates(int[] nums) {
		if (nums == null || nums.length == 0)
			return 0;
		int t = nums[0];
		int count = 0;
		int c = 0;
		for (int i = 1; i < nums.length - count; i++) {
			if (nums[i] == t) {
				c++;
				if (c > 1) {
					for (int j = i; j < nums.length - 1; j++) {
						nums[j] = nums[j + 1];
					}
					i--;
					count++;
				}
			} else {
				t = nums[i];
				c = 0;
			}
		}
		return nums.length - count;
	}
	```

 



## 114. 不同的路径
 ```java
public int uniquePaths(int m, int n) {
	if (m == 0 || n == 0) {
		return 1;
	}
	// 数组中存储到达第i行第j列的可能的路径数量和
	int[][] sum = new int[m][n];
	// 第一行和第一列的所有值都是1，因为到达这些位置只可能有一条路径
	for (int i = 0; i < m; i++) {
		sum[i][0] = 1;
	}
	for (int i = 0; i < n; i++) {
		sum[0][i] = 1;
	}
	// 到达网格的i行j列可能的路径为到达i-1行，j列的路径数加上到达i行，j-1列的路径数，因为机器人只有这两条途径能到达目的地[i][j]。
	for (int i = 1; i < m; i++) {
		for (int j = 1; j < n; j++) {
			sum[i][j] = sum[i - 1][j] + sum[i][j - 1];
		}
	}
	return sum[m - 1][n - 1];
}
 ```
## 115.不同的路径 II
```java
public int uniquePathsWithObstacles(int[][] obstacleGrid) {
    if (obstacleGrid == null || obstacleGrid.length == 0 || obstacleGrid[0].length == 0) {
        return 1;
    }
    int m = obstacleGrid.length;
    int n = obstacleGrid[0].length;
    int[][] sum = new int[m][n];
    for (int i = 0; i < m; i++) {
        // 遇到障碍，下面的都走不通了
        if (obstacleGrid[i][0] == 1) {
            break;
        }
        sum[i][0] = 1;
    }
    for (int i = 0; i < n; i++) {
        if (obstacleGrid[0][i] == 1) {
            break;
        }
        sum[0][i] = 1;
    }
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            if (obstacleGrid[i][j] != 1) {
                sum[i][j] = sum[i - 1][j] + sum[i][j - 1];
            } else {
                sum[i][j] = 0;
            }
        }
    }
    return sum[m - 1][n - 1];
}
```
## 117. 跳跃游戏 II
```java
public int jump(int[] A) {
    if (A.length <= 1) {
        return 0;
    }
    int step = 0;
    int index = 0;
    int i = 0;
    while (i < A.length) {
        if (i + A[i] >= A.length - 1) {
            return step + 1;
        }
        int max = -1;
        for (int j = i + 1; j <= i + A[i]; j++) {
            if (max < j + A[j]) {
                max = j + A[j];
                index = j;
            }
        }
        step++;
        i = index;
    }
    return step;
}
```
## 120. 单词接龙
	public boolean connect(String word1, String word2) {
		int count = 0;
		for (int i = 0; i < word1.length(); i++) {
			if (word1.charAt(i) != word2.charAt(i)) {
				count++;
			}
		}
		return count == 1;
	}
	public void constructGraph(String start, String end, Set<String> dict, Map<String, LinkedList> graph) {
		dict.add(start);
		dict.add(end);
		String[] dictArr = dict.toArray(new String[0]);
		for (int i = 0; i < dictArr.length; i++) {
			graph.put(dictArr[i], new LinkedList<String>());
		}
		for (int i = 0; i < dictArr.length; i++) {
			for (int j = i + 1; j < dictArr.length; j++) {
				if (connect(dictArr[i], dictArr[j])) {
					graph.get(dictArr[i]).add(dictArr[j]);
					graph.get(dictArr[j]).add(dictArr[i]);
				}
			}
		}
	}
	public int BFS(String start, String end, Set<String> dict, Map<String, LinkedList> graph) {
		Set<String> visit = new HashSet();
		Queue<String> queue = new LinkedList();
		queue.add(start);
		int currStep = 0;
		while (!queue.isEmpty()) {
			currStep++;
			int size = queue.size();
			for (int i = 0; i < size; i++) {
				String currWord = queue.poll();
				visit.add(currWord);
				if (currWord.equals(end)) {
					return currStep;
				} else {
					List<String> neighbor = graph.get(currWord);
					for (int j = 0; j < neighbor.size(); j++) {
						if (!visit.contains(neighbor.get(j))) {
							queue.add(neighbor.get(j));
						}
					}
				}
			}
		}
		return 0;
	}
	public int ladderLength(String start, String end, Set<String> dict) {
		if (dict == null) {
			return 0;
		} else if (start.equals(end)) {
			return 1;
		}
		Map<String, LinkedList> graph = new HashMap();
		constructGraph(start, end, dict, graph);
		int step = BFS(start, end, dict, graph);
		return step;
	}
## 123.单词搜索
	public boolean find(char[][] board, char[] str, int x, int y, int k) {
		if (k >= str.length) {
			return true;
		}
		if (x < 0 || x >= board.length || y < 0 || y >= board[0].length || board[x][y] != str[k]) {
			return false;
		}
	
		board[x][y] = '#';
		boolean rst = find(board, str, x - 1, y, k + 1) || find(board, str, x, y - 1, k + 1) || find(board, str, x + 1, y, k + 1) || find(board, str, x, y + 1, k + 1);
		board[x][y] = str[k];
		return rst;
	}
	
	public boolean exist(char[][] board, String word) {
		if (board == null || board.length == 0) {
			return false;
		}
		if (word.length() == 0) {
			return true;
		}
		char[] str = word.toCharArray();
		boolean ret = false;
		for (int i = 0; i < board.length; i++) {
			for (int j = 0; j < board[i].length; j++) {
				if (board[i][j] == str[0]) {
					ret = find(board, str, i, j, 0);
					if (ret) {// 返回值真直接返回，返回值假时继续执行
						return ret;
					}
				}
			}
		}
		return ret;
	}
## 124.最长连续序列
```java
public int longestConsecutive(int[] num) {
    Set<Integer> set = new HashSet<>();
    for (int item : num) {
        set.add(item);
    }
    int maxLen = 0;
    for (int item : num) {
        if (set.contains(item)) {
            set.remove(item);
            int left = item - 1;
            int right = item + 1;
            while (set.contains(left)) {
                set.remove(left);
                left--;
            }
            while (set.contains(right)) {
                set.remove(right);
                right++;
            }
            maxLen = Math.max(maxLen, right - left - 1);
        }
    }
    return maxLen;
}
```
## 127.拓扑排序
```java
public ArrayList<DirectedGraphNode> topSort(ArrayList<DirectedGraphNode> graph) {
    ArrayList<DirectedGraphNode> result = new ArrayList<DirectedGraphNode>();
    Map<DirectedGraphNode, Integer> map = new HashMap<>();
    for (DirectedGraphNode node : graph) {
        for (DirectedGraphNode neighbor : node.neighbors) {
            if (map.containsKey(neighbor)) {
                map.put(neighbor, map.get(neighbor) + 1);
            } else {
                map.put(neighbor, 1);
            }
        }
    }
    Queue<DirectedGraphNode> queue = new LinkedList<DirectedGraphNode>();
    for (DirectedGraphNode node : graph) {
        if (!map.containsKey(node)) {
            queue.offer(node);
            result.add(node);
        }
    }
    while (!queue.isEmpty()) {
        DirectedGraphNode node = queue.poll();
        for (DirectedGraphNode n : node.neighbors) {
            map.put(n, map.get(n) - 1);
            if (map.get(n) == 0) {
                result.add(n);
                queue.offer(n);
            }
        }
    }
    return result;
}
```
## 129. 重哈希
		public ListNode[] rehashing(ListNode[] hashTable) {
		if (hashTable == null || hashTable.length == 0) {
			return null;
		}
		int newCapacity = hashTable.length * 2;
		ListNode[] newTable = new ListNode[hashTable.length * 2];
		for (int i = 0; i < hashTable.length; i++) {
			ListNode node = hashTable[i];
			while (node != null) {
				int hashVal = (node.val % newCapacity + newCapacity) % newCapacity;
				ListNode p = newTable[hashVal];
				if (p == null) {
					newTable[hashVal] = new ListNode(node.val);
				} else {
					while (p.next != null) {
						p = p.next;
					}
					p.next = new ListNode(node.val);
				}
				node = node.next;
			}
		}
		return newTable;
	}
## 135.数字组合
	public List<List<Integer>> combinationSum(int[] num, int target) {
		List<List<Integer>> result = new ArrayList<>();
		if (num == null || num.length == 0) {
			return result;
		}
		Arrays.sort(num);
		int start = 0;
		List<Integer> item = new ArrayList<>();
		dfs(num, target, item, result, start);
		return result;
	}
	private void dfs(int[] num, int target, List<Integer> item, List<List<Integer>> result, int start) {
		if (target == 0) {
			result.add(new ArrayList(item));
			return;
		}
		for (int i = start; i < num.length; i++) {
			if (i != start && num[i] == num[i - 1]) {
				continue;
			}
			if (num[i] <= target) {
				item.add(num[i]);
				target -= num[i];
				dfs(num, target, item, result, i);
				target += num[i];
				item.remove(item.size() - 1);
			}
		}
	}
## 136.分割回文串
```java
public List<List<String>> partition(String s) {
    List<List<String>> results = new ArrayList<>();
<<<<<<< HEAD
    if (s == null || s.length() == 0) {
        return results;
    }
=======
>>>>>>> 修改
    List<String> item = new ArrayList<String>();
    dfs(s, 0, item, results);
    return results;
}

private void dfs(String s, int startIndex, List<String> item, List<List<String>> results) {
    if (startIndex == s.length()) {
        results.add(new ArrayList<String>(item));
        return;
    }
    for (int i = startIndex; i < s.length(); i++) {
        String subString = s.substring(startIndex, i + 1);
        if (!isPalindrome(subString)) {
            continue;
        }
        item.add(subString);
        dfs(s, i + 1, item, results);
        item.remove(item.size() - 1);
    }
}

private boolean isPalindrome(String s) {
    for (int i = 0, j = s.length() - 1; i < j; i++, j--) {
        if (s.charAt(i) != s.charAt(j)) {
            return false;
        }
    }
    return true;
}
```
## 138. 子数组之和
```java
public List<Integer> subarraySum(int[] nums) {
    List<Integer> pos = new ArrayList<Integer>(2);
    if (nums == null || nums.length == 0) {
        return pos;
    }
    int sum = 0;
    Map<Integer, Integer> map = new HashMap<>();
    map.put(sum, -1);
    for (int i = 0; i < nums.length; i++) {
        // 从某个位置出发，到i的sum与到j的sum相等，说明从i+1到j的sum为零。
        sum += nums[i];
        if (map.containsKey(sum)) {
            pos.add(map.get(sum) + 1);
            pos.add(i);
            break;
        }
        map.put(sum, i);
    }
    return pos;
}
```
## 139. 最接近零的子数组和
	class Pair {
		int sum;
		int index;
	
		public Pair(int s, int i) {
			sum = s;
			index = i;
		}
	}
	
	public int[] subarraySumClosest(int[] nums) {
		int[] result = new int[2];
		if (nums == null || nums.length <= 1) {
			return result;
		}
		//sum[i]表示0到i位置的和，那么求sum[j]-sum[i]就可以求得子数组和
		Pair[] sums = new Pair[nums.length + 1];
		sums[0] = new Pair(0, 0);
		for (int i = 1; i < sums.length; i++) {
			sums[i] = new Pair(sums[i - 1].sum + nums[i - 1], i);
		}
		Arrays.sort(sums, new Comparator<Pair>() {
			public int compare(Pair a, Pair b) {
				return a.sum - b.sum;
			}
		});
		int ans = Integer.MAX_VALUE;
		for (int i = 1; i < sums.length; i++) {
			if (ans > sums[i].sum - sums[i - 1].sum) {
				ans = sums[i].sum - sums[i - 1].sum;
				result[0] = Math.min(sums[i].index, sums[i - 1].index);
				result[1] = Math.max(sums[i].index, sums[i - 1].index) - 1;
			}
		}
		return result;
	}
## 140. 快速幂
	思路：每次二分n，然后递归的去求a^n % b。可以分为两种情况： 
	1.如果n为奇数可以转化为(a^(n/2)*a^(n/2)*a)%b
	2.如果n为偶数可以转化为(a^(n/2)*a^(n/2)) %b
	取模运算的乘法法则： (a * b) % p = (a % p * b % p) % p
	而且a^1 = a , a^0=1，这样我们的实际的时间复杂度是O(log(n))。
	public int fastPower(int a, int b, int n) {
		if (n == 0) {
			return 1 % b;
		} else if (n == 1) {
			return a % b;
		}
		int result = fastPower(a, b, n / 2);
		if (n % 2 == 1) {
			result = (result * result % b) * a % b;
		} else {
			result = result * result % b;
		}
		return (int) result;
	}
## 141. x的平方根
	public int sqrt(int x) {
		int left = 0;
		int right = x;
		while (left <= right) {
			int mid = (left + right) / 2;
			if (Math.pow(mid, 2) > x) {
				right = mid - 1;
			} else {
				left = mid + 1;
			}
		}
		return left - 1;
	}
## 142. O(1)时间检测2的幂次
	public boolean checkPowerOf2(int n) {
	     if (n <= 0) {
	        return false;
	    }
	    return (n & (n-1)) == 0;
	}
## 144. 交错正负数
	public void rerange(int[] A) {
		if (A == null || A.length <= 1) {
			return;
		}
		Arrays.sort(A);
		int i, j;
		if (A.length % 2 == 0) {
			i = 1;
			j = A.length - 2;
		} else if (A[A.length / 2] > 0) {
			i = 0;
			j = A.length - 2;
		} else {
			i = 1;
			j = A.length - 1;
		}
		while (i < j) {
			swap(A, i, j);
			i += 2;
			j -= 2;
		}
	}
	private void swap(int[] A, int i, int j) {
		int t = A[i];
		A[i] = A[j];
		A[j] = t;
	}

## 152. 组合
	public List<List<Integer>> combine(int n, int k) {
		List<List<Integer>> result = new ArrayList<>();
		if (n == 0 || k == 0) {
			return result;
		}
		List<Integer> item = new ArrayList();
		dfs(n, k, item, result, 1);
	
		return result;
	}
	public void dfs(int n, int k, List<Integer> item, List<List<Integer>> result, int start) {
		if (item.size() == k) {
			result.add(new ArrayList(item));
			return;
		}
		for (int i = start; i <= n; i++) {
			item.add(i);
			dfs(n, k, item, result, i + 1);
			item.remove(item.size() - 1);
		}
	}
## 153. 数字组合 II
	public List<List<Integer>> combinationSum2(int[] num, int target) {
		List<List<Integer>> result = new ArrayList<>();
		if (num == null || num.length == 0) {
			return result;
		}
		Arrays.sort(num);
	
		int start = 0;
		List<Integer> item = new ArrayList<>();
		dfs(num, target, item, result, start);
	
		return result;
	}
	
	private void dfs(int[] num, int target, List<Integer> item, List<List<Integer>> result, int start) {
		if (target == 0) {
			result.add(new ArrayList(item));
			return;
		}
		for (int i = start; i < num.length; i++) {
			if (i != start && num[i] == num[i - 1]) {
				continue;
			}
			if (num[i] <= target) {
				item.add(num[i]);
				target -= num[i];
				dfs(num, target, item, result, i + 1);
				target += num[i];
				item.remove(item.size() - 1);
			}
		}
	}
## 156.合并区间
	private class IntervalComparator implements Comparator<Interval> {
		@Override
		public int compare(Interval a, Interval b) {
			return a.start - b.start;
		}
	}
	
	public List<Interval> merge(List<Interval> intervals) {
		if (intervals == null || intervals.size() <= 1) {
			return intervals;
		}
		List<Interval> list = new ArrayList<>();
		Collections.sort(intervals, new IntervalComparator());
	
		list.add(intervals.get(0));
		for (int i = 1; i < intervals.size(); i++) {
			Interval pre = list.get(list.size() - 1);
			Interval curr = intervals.get(i);
			if (curr.start > pre.end) {
				list.add(curr);
			} else if (curr.start <= pre.end && curr.end > pre.end) {
				list.get(list.size() - 1).end = curr.end;
			}
		}
		return list;
	}
## 159. 寻找旋转排序数组中的最小值
```java
/**
把一个排序数组先分割成两部分[first, second]，其中，first代表前面几个元素，second代表之后的元素， 例如对于数组[0, 1, 2, 4, 5, 6, 7]，可以设定first = [0, 1, 2], second = [4, 5, 6, 7]. 那么经过旋转之后，数组就变成了[second, first]，我们观察一下，这个新数组有这样两个特性：（1）second中所有元素都大于first中任意元素（2）second与first都是递增的序列
*/
public int findMin(int[] nums) {
    int left = 0;
    int right = nums.length - 1;

    while (left < right && nums[left] > nums[right]) {
         int mid = (left + right) / 2;
        if (nums[left] < nums[mid]) { //mid指在second中，而最小值肯定在mid后面
            left = mid + 1;
        } else {//mid指在first中
            right = mid;
        }
    }
    return nums[left];
}
```

## 160. 寻找旋转排序数组中的最小值（有重复数字）
```java
public int findMin(int[] nums) {
    int low = 0;
    int high = nums.length - 1;
    while (low < high && nums[low] >= nums[high]) {
        int mid = (low + high) / 2;
        if (nums[low] < nums[mid]) {
            low = mid + 1;
        } else if (nums[low] > nums[mid]) {
            high = mid;
        } else {
            low = low + 1;
        }
    }
    return nums[low];
}
```



## 169. 汉诺塔
	public List<String> towerOfHanoi(int n) {
		List<String> list = new ArrayList();
		if (n <= 0) {
			return list;
		}
		hanoi(n, 'A', 'B', 'C', list);
		return list;
	}
	void hanoi(int n, char A, char B, char C, List<String> list) {
		if (n == 1) {
			list.add("from " + A + " to " + C);
		} else {
			hanoi(n - 1, A, C, B, list);
			list.add("from " + A + " to " + C);
			hanoi(n - 1, B, A, C, list);
		}
	}

## 171. 乱序字符串
	public List<String> anagrams(String[] strs) {
		HashMap<String, ArrayList> map = new HashMap<>();
		ArrayList<String> ret = new ArrayList<String>();
		for (int i = 0; i < strs.length; i++) {
			char[] ch = strs[i].toCharArray();
			Arrays.sort(ch);
			String str = String.valueOf(ch);
	
			ArrayList<String> list = map.get(str);
			if (list == null) {
				list = new ArrayList<>();
			}
			list.add(strs[i]);
			map.put(str, list);
		}
		for (String str : map.keySet()) {
			ArrayList<String> list = map.get(str);
			if (list.size() != 1) {
				ret.addAll(list);
			}
		}
		return ret;
	}
## 172. 删除元素
	public int removeElement(int[] A, int elem) {
		if (A == null || A.length == 0) {
			return 0;
		}
		int count = 0;
		int len = 0;
		for (int i = 0; i < A.length; i++) {
			if (A[i] == elem) {
				len++;
				count++;
			} else if (count > 0 && count < A.length) {
				for (int j = i; j < A.length; j++) {
					A[j - count] = A[j];
				}
				i -= count;
				count = 0;
			}
		}
		return A.length - len;
	}



## 176 图中两个点之间的路线 PASS

## 178. 图是否是树
```java
public boolean validTree(int n, int[][] edges) {
    if (n == 0) {
        return false;
    } else if (edges.length != n - 1) {
        return false;
    }
    Map<Integer, Set<Integer>> graph = initializeGraph(n, edges);
    Queue<Integer> queue = new LinkedList<>();
    Set<Integer> hash = new HashSet<>();

    queue.offer(0);
    hash.add(0);
    while (!queue.isEmpty()) {
        int node = queue.poll();
        for (Integer neighbor : graph.get(node)) {
            if (!hash.contains(neighbor)) {
                hash.add(neighbor);
                queue.offer(neighbor);
            }
        }
    }
    return (hash.size() == n);
}

private Map<Integer, Set<Integer>> initializeGraph(int n, int[][] edges) {
    Map<Integer, Set<Integer>> graph = new HashMap<>();
    for (int i = 0; i < n; i++) {
        graph.put(i, new HashSet<Integer>());
    }
    for (int i = 0; i < edges.length; i++) {
        int u = edges[i][0];
        int v = edges[i][1];
        graph.get(u).add(v);
        graph.get(v).add(u);
    }
    return graph;
}
```
## 181. 将整数A转换为B
	public int bitSwapRequired(int a, int b) {
		int count = 0;
		for (int c = a ^ b; c != 0; c = c >>> 1) {
			count += c & 1;
		}
		return count;
	}
## 182. 删除数字
	public String DeleteDigits(String A, int k) {
		StringBuffer sb = new StringBuffer(A);
		for (int i = 0; i < k; i++) {
			int j = 0;
			while (j < sb.length() - 1 && sb.charAt(j) <= sb.charAt(j + 1)) {
				j++;
			}
			sb.delete(j, j + 1);
		}
		while (sb.length() > 1 && sb.charAt(0) == '0') {
			sb.delete(0, 1);
		}
		return sb.toString();
	}
## 184. 最大数
	private class NumComparator implements Comparator<String> {
		@Override
		//该Str1按字典顺序小于参数字符串Str2，则返回值小于0；若Str1按字典顺序大于参数字符串Str2，则返回值大于0
		//如果没有字符不同，compareTo 返回这两个字符串长度的差
		public int compare(String s1, String s2) {
			return (s2 + s1).compareTo(s1 + s2);
		}
	}
	public String largestNumber(int[] nums) {
		if (nums == null || nums.length == 0) {
			return "";
		}
		String[] strArr = new String[nums.length];
		for (int i = 0; i < nums.length; i++) {
			strArr[i] = String.valueOf(nums[i]);
		}
		Arrays.parallelSort(strArr, new NumComparator());
		StringBuilder sb = new StringBuilder(String.valueOf(0));
		for (int i = 0; i < strArr.length; i++) {
			if (sb.length() == 1 && !strArr[i].equals("0")) {
				continue;
			}
			sb.append(strArr[i]);
		}
		return sb.length() > 1 ? sb.toString().substring(1) : sb.toString();
	}
## 189. 丢失的第一个正整数
	public int firstMissingPositive(int[] A) {
	    if (A == null) {
	        return 1;
	    }
	    for (int i = 0; i < A.length; i++) {
	        while (A[i] > 0 && A[i] <= A.length && A[i] != (i+1)) {
	            int tmp = A[A[i]-1];
	            if (tmp == A[i]) {
	                break;
	            }
	            A[A[i]-1] = A[i];
	            A[i] = tmp;
	        }
	    }
	    for (int i = 0; i < A.length; i ++) {
	        if (A[i] != i + 1) {
	            return i + 1;
	        }
	    }
	    return A.length + 1;
	}
## 196.Missing Number
	public int findMissing(int[] nums) {
		if (nums == null || nums.length == 0) {
			return 0;
		}
		int i = 0;
		while (i < nums.length) {
			int num = nums[i];
			if (num < nums.length && num != nums[num]) {
				int t = nums[i];
				nums[i] = nums[num];
				nums[num] = t;
			} else {
				i++;
			}
		}
		for (i = 0; i < nums.length; i++) {
			if (i != nums[i]) {
				return i;
			}
		}
		return nums.length;
	}
## 384. 最长无重复字符的子串

给定一个字符串，请你找出其中不含有重复字符的 **最长子串** 的长度。

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> map = new HashMap<Character, Integer>();
    int longest = -1;
    for (int i = 0; i < s.length(); i++) {
        char ch = s.charAt(i);
        if (map.containsKey(ch)) {
            longest = Math.max(map.size(), longest);
            i = map.get(ch); //回退
            map.clear();
        } else {
            map.put(ch, i);
        }
    }
    longest = Math.max(map.size(), longest);
    return longest;
}
```
## 384. 最长无重复字符的子串
```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> map = new HashMap<Character, Integer>();
    
    int longest = -1;
    for (int i = 0; i < s.length(); i++) {
        char ch = s.charAt(i);
        if (map.containsKey(ch)) {
            longest = Math.max(map.size(), longest);
            i = map.get(ch);
            map.clear();
        } else {
            map.put(ch, i);
        }
    }
    longest = Math.max(map.size(), longest);
    return longest;
}
```
## 433. 岛屿的个数
	public void DFS(boolean[][] grid, boolean[][] mark, int x, int y) {
		mark[x][y] = true; // 标记已搜索的位置
		final int[] dx = { -1, 1, 0, 0 }; // 方向数组
		final int[] dy = { 0, 0, -1, 1 };
		for (int i = 0; i < 4; i++) {
			int newx = dx[i] + x;
			int newy = dy[i] + y;
			if (newx < 0 || newx >= grid.length || newy < 0 || newy >= grid.length) {
				continue;
			}
			if (grid[newx][newy] && !mark[newx][newy]) {
				DFS(grid, mark, newx, newy);
			}
		}
	}
	
	public void BFS(boolean[][] grid, boolean[][] mark, int x, int y) {
		mark[x][y] = true; // 标记已搜索的位置
		final int[] dx = { -1, 1, 0, 0 }; // 方向数组
		final int[] dy = { 0, 0, -1, 1 };
	
		Queue<int[]> queue = new LinkedList();
		queue.add(new int[] { x, y });
		while (!queue.isEmpty()) {
			int[] x_y = queue.poll();
			x = x_y[0];
			y = x_y[1];
			for (int i = 0; i < 4; i++) {
				int newx = dx[i] + x;
				int newy = dy[i] + y;
				if (newx < 0 || newx >= grid.length || newy < 0 || newy >= grid[newx].length) {
					continue;
				}
				if (!mark[newx][newy] && grid[newx][newy]) {
					queue.offer(new int[] { newx, newy });
					mark[newx][newy] = true;
				}
			}
		}
	}
	
	public int numIslands(boolean[][] grid) {
		if (grid == null || grid.length == 0) {
			return 0;
		}
		int count = 0;
		boolean[][] mark = new boolean[grid.length][grid[0].length];
	
		for (int i = 0; i < grid.length; i++) {
			for (int j = 0; j < grid[i].length; j++) {
				if (grid[i][j] && !mark[i][j]) {
					BFS(grid, mark, i, j);
					count++;
				}
			}
		}
		return count;
	}
## 1220. 用火柴摆正方形
能否使用所有火柴摆成一个正方形。每个火柴必须使用一次，所有火柴都得用上，火柴不应该被打破。     

```java
public boolean makesquare(int[] nums) {
	if (nums.length < 4) {
		return false;
	}
	int sum = 0;
	for (int num : nums) {
		sum += num;
	}
	if (sum % 4 != 0) {
		return false;
	}
	Arrays.sort(nums);
	int[] bucket = new int[4];
	return generate(0, sum / 4, nums, bucket);

}
private boolean generate(final int i, int target, int[] nums, int[] bucket) {
	if (i >= nums.length) {
		return bucket[0] == target && bucket[1] == target && bucket[2] == target && bucket[3] == target;
	}
	for (int j = 0; j < bucket.length; j++) {
		if ((bucket[j] + nums[i]) > target) {
			continue;
		}
		bucket[j] += nums[i];
		if (generate(i + 1, target, nums, bucket)) {
			return true;
		}
		bucket[j] -= nums[i];
	}
	return false;
}
```