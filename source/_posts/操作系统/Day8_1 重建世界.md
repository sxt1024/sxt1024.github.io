```

void putint(int ch);

#define DATA_BASE 0x60000

void putint(int num)
{
    int i = 0;
    char *chs = (char *)DATA_BASE; //need point mem alloc
    do
    {
     
        i++;

    } while ((num /= 10) > 0); //删除该数字
    for (int j = i; j >= 0; j--)
    { //生成的数字是逆序的，所以要逆序输出
    
       *(chs + i) = (char)(num % 10 + '0'); //取下一个数字
        putchar(*(chs + j));
        *(chs + j) = 0;
    }
}



```

引申出内存管理的问题