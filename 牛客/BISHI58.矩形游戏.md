- 质数的因数是1和他自己，而质因数只有他自己

```c++
#include <iostream>
using namespace std;

int minzhi(int n){
    for(int i = 2; i * i <= n; i++){
        if(n % i == 0){
            return i;
        }
    }
    //如果找不到最小的质因数的话，说明n本身就是质数，只能被1和自己整除；再由于a>1，所以只能返回自己
    return n;
}

int main() {
    
    int n;
    cin >> n;

    //最后一轮一定是1，这里直接给他加上，因为后面的while循环处理不了这个1
    int sum_goal = 1;

    while(n > 1){
        sum_goal += n;

        //a是n的最小质因数，这样就可以使b保持最大
        int a = minzhi(n);
        int b = n / a;
        n = b;
    }

    cout << sum_goal << endl;

    return 0;
}
```

