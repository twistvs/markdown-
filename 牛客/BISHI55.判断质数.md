1. 质数一定是大于1的数，所以<=1的数一定不是质数
2. 

```c++
#include <iostream>
using namespace std;
#include <vector>

int main() {
    long long n;
    cin >> n;
    
    //<=1的数不是质数
    if(n <= 1){
        cout << "No" << endl;
        return 0;
    }

    for(long long i = 2; i * i <= n; i++){
        //相比于分解质因数，只要n可以被某个i整除，那么他就不止可以被1和自己整除，那么就一定不是质数，直接跳出循环
        if(n % i == 0){
            cout << "No" << endl;
            return 0;
        }
    }

    //如果n可以顺利度过循环，说明没有数可以整除他，那他就是质数
    cout << "Yes" << endl;
    return 0;
}
```

