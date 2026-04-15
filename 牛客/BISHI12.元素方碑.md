1. 由于到最后所有方碑的值都必须一样，所以能量的总值除以方碑数量必须能够整除
2. 由于每次操作都是在相隔一个方碑的两个方碑上操作，也就是说能量只在同为奇或者同为偶的两个方碑上流动，也就是说奇数上的总能量要等于平均值乘以奇数的数量；偶数同理
3. 在这里，我们通过计算一个ji变量，他是所有奇数能量-avg的总和，只要这个变量最终为0，那么奇数就是可以实现的，然后再判断偶数

```c++
#include <iostream>
using namespace std;
#include <vector>

int main() {
    int times;
    cin >> times;

    //案例循环，一共times个案例
    for(int time = 1; time <= times; time++){
        int count;
        cin >> count;

        vector<int> energy(count, 0);
        int sum_energy = 0;
        for(int i = 0; i < count; i++){
            int a;
            cin >> a;
            energy[i] = a;
            sum_energy += a;
        }
	
        //如果总能量不能整除方碑总数，直接输出no
        if(sum_energy%count != 0){
            cout << "NO" << endl;
            continue;
        }
        else{
            int avg = sum_energy/count;
            int ji = 0;
            int ou = 0;
		
            for(int i = 0; i < count; i++){
                //除了0之外，其他的偶数可以用i%2==0进行判断
                if(i == 0 || i%2 == 0){
                    ou += energy[i] - avg;
                }
                if(i%2 != 0){
                    ji += energy[i] - avg;
                }
            }
            //奇和偶必须同时为0
            if(ou == 0 && ji == 0){
                cout << "YES" << endl;
            }
            else{
                cout << "NO" << endl;
            }
        }
    }
    return 0;
}
```

