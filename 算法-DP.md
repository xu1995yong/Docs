## 392. 打劫房屋
假设你是一个专业的窃贼，准备沿着一条街打劫房屋。每个房子都存放着特定金额的钱。你面临的唯一约束条件是：相邻的房子装着相互联系的防盗系统，且 当相邻的两个房子同一天被打劫时，该系统会自动报警。

给定一个非负整数列表，表示每个房子中存放的钱， 算一算，如果今晚去打劫，你最多可以得到多少钱 在不触动报警装置的情况下。

```java	
public long houseRobber(int[] A) {
    if (A == null || A.length == 0) {
        return 0;
    } else if (A.length == 1) {
        return A[0];
    } else {
        long[] dp = new long[A.length];
        dp[0] = A[0];
        dp[1] = Math.max(dp[0], A[1]);
        for (int i = 2; i < A.length; i++) {
            dp[i] = Math.max(dp[i - 2] + A[i], dp[i - 1]);
        }
        return dp[A.length - 1];
    }
}
```



## 534. 打劫房屋 II
在上次打劫完一条街道之后，窃贼又发现了一个新的可以打劫的地方，但这次所有的房子围成了一个圈，这就意味着第一间房子和最后一间房子是挨着的。每个房子都存放着特定金额的钱。你面临的唯一约束条件是：相邻的房子装着相互联系的防盗系统，且 当相邻的两个房子同一天被打劫时，该系统会自动报警。

给定一个非负整数列表，表示每个房子中存放的钱， 算一算，如果今晚去打劫，你最多可以得到多少钱 在不触动报警装置的情况下。
```java
public int houseRobber2(int[] nums) {
    if (nums == null || nums.length == 0)
        return 0;
    if (nums.length == 1)
        return nums[0];
    if (nums.length == 2)
        return Math.max(nums[0], nums[1]);
    if (nums.length == 3)
        return Math.max(Math.max(nums[0], nums[1]), nums[2]);
    int len = nums.length;
    int res1 = houseRobber(nums, 0, len - 2);
    int res2 = houseRobber(nums, 1, len - 1);
    return Math.max(res1, res2);
}

public int houseRobber(int[] A, int start, int end) {
    if (start == end) {
        return A[start];
    } else {
        int[] dp = new int[end - start + 1];
        dp[0] = A[start];
        dp[1] = Math.max(dp[0], A[start + 1]);
        for (int i = 2; i <= end - start; i++) {
            dp[i] = Math.max(dp[i - 2] + A[start + i], dp[i - 1]);
        }
        return dp[end - start];
    }
}
```