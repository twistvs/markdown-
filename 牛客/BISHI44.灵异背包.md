#### 贪心

- 为了让和为偶数并且最大，所有偶数都可以直接加进去，因为偶数不可能会让偶变成奇
- 所有奇数加起来如果是偶，那就全部加，如果不是偶，就把最小的奇数去掉

```c++
#include <climits>
#include <iostream>
using namespace std;

int main() {
    int size;
    cin >> size;

    int ou_sum = 0;
    int ji_sum = 0;
    int min_ji = INT_MAX;
    for(int i = 0; i < size; i++){
        int a;
        cin >> a;
        if(a == 0 || a%2 == 0){
            ou_sum += a;
        }
        else{
            ji_sum += a;
            min_ji = min(min_ji, a);
        }
    }
    if(ji_sum%2 != 0){
        ji_sum -= min_ji;
    }
    cout << ou_sum + ji_sum << endl;

    return 0;
}
```

