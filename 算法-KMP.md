## 1. 暴力匹配算法
思路：    

假设现在文本串S匹配到 i 位置，模式串P匹配到 j 位置，则有：  

-   如果当前字符匹配成功（即S[i] == P[j]），则i++，j++，继续匹配下一个字符；
-	如果匹配失败（即S[i]! = P[j]），令i = i - (j - 1)，j = 0。相当于每次匹配失败时，i 回溯，j 被置为0。    


```c++
int ViolentMatch(char* s, char* p)  
{  
    int sLen = strlen(s);  
    int pLen = strlen(p);  

    int i = 0;  
    int j = 0;  
    while (i < sLen && j < pLen)  
    {  
        if (s[i] == p[j])  
        {  
            //①如果当前字符匹配成功（即S[i] == P[j]），则i++，j++      
            i++;  
            j++;  
        }  
        else  
        {  
            //②如果失配（即S[i]! = P[j]），令i = i - (j - 1)，j = 0     
            i = i - j + 1;  
            j = 0;  
        }  
    }  
    //匹配成功，返回模式串p在文本串s中的位置，否则返回-1  
    if (j == pLen)  
        return i - j;  
    else  
        return -1;  
}  
```

## 2. KMP算法

常用于在一个文本串S内查找一个模式串P 的出现位置