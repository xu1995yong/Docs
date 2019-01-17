## 描述：给出一个包含大小写字母的字符串。求出由这些字母构成的最长的回文串的长度是多少。	
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

