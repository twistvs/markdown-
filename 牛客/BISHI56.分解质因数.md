- **质因数**：若正整数p仅能被1与自身整除，则p为**质数**；若质数p能整除n，则p是n的**质因数**

```c++
#include <iostream>
using namespace std;

int main() {
    long long n;
    cin >> n;
 	
    //分解质因数的标准模板，要背住
    for (long long i = 2; i * i <= n; ++i) {
        while (n % i == 0) {
            cout << i << " ";
            n /= i;
        }
    }
 
    if (n > 1) {
        cout << n << " ";
    }
    cout << endl;
 
    return 0;
}
```

