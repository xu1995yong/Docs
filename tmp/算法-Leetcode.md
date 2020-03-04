## 求最长回文子串长度
```java
public int longestPalindrome(String s) {
    int retVal = 0;
    if (s == null | s.length() == 0) {
        return retVal;
    }
    int[] hash = new int[128];
    char[] ch = s.toCharArray();
    for (char i = 0; i < s.length(); i++) {
        int index = Integer.valueOf(ch[i]);
        hash[index]++;
    }
    // 是否有中心点
    boolean flag = false;

    for (char i = 0; i < hash.length; i++) {
        int count = hash[i];
        // 出现双数次
        if (count % 2 == 0) {
            retVal += count;
        } else {
            // 出现单数次
            retVal += (count - 1);
            flag = true;
        }
    }
    if (flag){
        retVal += 1;
    }
    return retVal;
}
```

## 求最长回文子串

```java
public String longestPalindrome(String str) {
    if (str.length() < 2) {
        return str;
    }
    char chs[] = str.toCharArray();
    boolean[][] dp = new boolean[chs.length][chs.length];
    dp[chs.length - 1][chs.length - 1] = true;

    int start = 0;
    int maxLen = 1;

    for (int i = chs.length - 2; i >= 0; i--) {
        dp[i][i] = true;
        for (int j = chs.length - 1; j >= i; j--) {
            if (j == i + 1) {
                dp[i][j] = chs[i] == chs[j];
            } else if (j > i + 1) {
                dp[i][j] = (chs[i] == chs[j]) && dp[i + 1][j - 1];
            }
            if (maxLen < j - i + 1 && dp[i][j]) {
                start = i;
                maxLen = j - i + 1;
            }
        }
    }
    return str.substring(start, start + maxLen);
}
```





## 给定一个以字符串表示的非负整数 num，移除这个数中的 k 位数字，使得剩下的数字最小。

```java
public String removeKdigits(String num, int k) {
    if (num.length() == 0)
        return "";
    if (num.length() <= k) {
        return "0";
    }
    while (k > 0 && num.length() != 0) {
        boolean flag = false;
        for (int i = 0; i < num.length() - 1; i++) {
            int a = Integer.valueOf(num.charAt(i));
            int b = Integer.valueOf(num.charAt(i + 1));
            if (a > b) {
                num = num.substring(0, i) + num.substring(i + 1, num.length());
                flag = true;
                break;
            }
        }
        if (!flag) {
            num = num.substring(0, num.length() - 1);

        }
        while (num.length() != 1 && num.charAt(0) == '0') {
            num = num.substring(1);
        }
        k--;
    }
    return num;
}
```

## 6. Z 字形变换

将一个给定字符串根据给定的行数，以从上往下、从左到右进行 Z 字形排列。
比如输入字符串为 `"LEETCODEISHIRING"` 行数为 3 时，排列如下：

```
L   C   I   R
E T O E S I I G
E   D   H   N
```
之后，你的输出需要从左往右逐行读取，产生一个新的字符串，比如：`"LCIRETOESIIGEDHN"`

```java
public String convert(String s, int numRows) {
    if (numRows == 1) return s;
    List<StringBuilder> rows = new ArrayList<>();
    for (int i = 0; i < Math.min(numRows, s.length()); i++) {
        rows.add(new StringBuilder());
    }

    int curRow = 0;
    boolean goingDown = false;
    for (char c : s.toCharArray()) {//将每个字符添加到所属行的stringbuilder中
        rows.get(curRow).append(c);
        if (curRow == 0 || curRow == numRows - 1) {
            goingDown = !goingDown;
        }
        curRow += goingDown ? 1 : -1;
    }

    StringBuilder ret = new StringBuilder();
    for (StringBuilder row : rows) {
        ret.append(row);
    }
    return ret.toString();
}
```

## 11. 盛最多水的容器

```java
public int maxArea(int[] height) {
//容器的盛水量取决于容器的底和容器较短的那条高。可见只有较短边会对盛水量造成影响，因此
//移动较短边的指针，并比较当前盛水量和当前最大盛水量。直至左右指针相遇。可以证明，如果
//移动较高的边，则盛水量只会变少；移动较低的边，则可以遍历到最大的情况。
    int left = 0;
    int right = height.length - 1;
    int max = 0;
    while (left < right) {
        max = Math.max(max, Math.min(height[left], height[right]) * (right - left));
        if (height[left] < height[right]) {
            left++;
        } else {
            right--;
        }
    }
    return max;
}
```



## 43. 

```java
public String multiply(String num1, String num2) {
    if (num1.equals("0") || num2.equals("0")) {
        return "0";
    }
    char[] nums1 = num1.toCharArray();
    char[] nums2 = num2.toCharArray();
    int[] val = new int[nums1.length + nums2.length];

    for (int j = nums2.length - 1; j > -1; j--) {
        int v1 = nums2[j] - '0';
        for (int i = nums1.length - 1; i > -1; i--) {
            int v2 = nums1[i] - '0';
            long temp = v1 * v2;
            int k = i + j + 1;
            while (temp != 0) {
                val[k] += temp % 10;
                temp /= 10;
                k--;
            }
        }
    }
    StringBuffer sb = new StringBuffer();
    for (int i = val.length - 1; i > -1; i--) {
        if (val[i] > 9) {
            val[i - 1] += val[i] / 10;
            val[i] %= 10;
        }
    }

    for (int i = 0; i < val.length; i++) {
        if ((i == 0 && val[0] != 0) || i != 0) {
            sb.append(val[i]);
        }
    }
    return sb.toString();
}
```

<<<<<<< HEAD
## 49. 字母异位词分组

给定一个字符串数组，将字母异位词组合在一起。字母异位词指字母相同，但排列不同的字符串。

```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List> ans = new HashMap<>();
    int[] count = new int[26];
    for (String s : strs) {
        Arrays.fill(count, 0);
        for (char c : s.toCharArray()) {
            count[c - 'a']++;
        }
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 26; i++) {
            sb.append('#');
            sb.append(count[i]);
        }
        String key = sb.toString();
        if (!ans.containsKey(key)) {
            ans.put(key, new ArrayList());
        }
        ans.get(key).add(s);
    }
    return new ArrayList(ans.values());
}
```

=======
>>>>>>> 修改
## 88.  合并两个有序数组

```java
public void merge(int[] nums1, int m, int[] nums2, int n) {
    int j = 0;
    for (int i = 0; i < nums1.length && j < nums2.length; i++) {
        if (nums1[i] > nums2[j]) {
            for (int k = m; k > i; k--) {
                nums1[k] = nums1[k - 1];
            }
            nums1[i] = nums2[j];
            j++;
            m++;
        }
    }
    while (j != nums2.length) {
        nums1[m] = nums2[j];
        m++;
        j++;
    }
}
```

<<<<<<< HEAD
## [55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game/)

```java
public boolean canJump(int[] nums) {
    int right = nums.length - 1;
    int i = right;
    while (i >= 0) {
        if (nums[i] >= right - i) {    //如果当前下标可以跳转到right，就更新right的值
            right = i;
        }
        i--;
    }
    if (right != 0) return false;    //当整个循环结束时，right没有到达0,说明不可抵达
    return true;
=======
## 322. 零钱兑换
给定不同面额的硬币 coins 和一个总金额 amount。编写一个函数来计算可以凑成总金额所需的最少的硬币个数。如果没有任何一种硬币组合能组成总金额，返回 -1。

```java
public int coinChange(int[] coins, int amount) {
    //dp[i]代表当金额为i时，凑成该金额的最少零钱数
    int[] dp = new int[amount + 1];
    //初始化
    for (int i = 0; i < dp.length; i++) {
        dp[i] = -1;
    }
    dp[0] = 0;
    for (int i = 1; i < dp.length; i++) {
        for (int j = 0; j < coins.length; j++) {//循坏各个面值的金额，
            //只有当金额i比零钱金额coins[j]大或相等时，
            // i - coins[j] 代表
            if (i - coins[j] >= 0 && dp[i - coins[j]] != -1) {
                if (dp[i] == -1 || dp[i] > dp[i - coins[j]] + 1) {
                    dp[i] = dp[i - coins[j]] + 1;
                }
            }
        }
    }
    return dp[amount];
>>>>>>> 修改
}
```



<<<<<<< HEAD
## 670. 最大交换

```java
public int maximumSwap(int num) {
    char[] digits = Integer.toString(num).toCharArray();
    if(digits.length == 1) return num;

    // 寻找不符合非递增顺序的分界线
    int split = 0;
    for (int i = 0; i < digits.length-1; i++){
        if (digits[i] < digits[i+1]){
            split = i+1;
            break;
        }
    }

    // 在分界线后面的部分寻找最大的值max
    char max = digits[split];
    int index1 = split;
    for (int j = split+1; j < digits.length; j++){
        if (digits[j] >= max){
            max = digits[j];
            index1 = j;
        }
    }

    // 在分界线前面的部分向前寻找小于max的最大值
    int index2 = split;
    for (int i = split-1; i >= 0; i--){
        if (digits[i] >= max){
            break;
        }
        index2--;
    }

    //交换两位找到的char
    char temp = digits[index1];
    digits[index1] = digits[index2];
    digits[index2] = temp;

    return Integer.valueOf(new String(digits));
}
```

=======
>>>>>>> 修改
