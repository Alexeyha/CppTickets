# ����� ��������

## ��������� �����������

� C++ �� ��������� ��� ������������ ������� / 
�������� ������� � ������� ��� �������� �� ������� ���������, ��� ������ ����� ���������� � 
���������, ������������ � ������ �� �������� ��� ������.

��� ���� ���������� ������ ������� ������ �� ������, �� ���� �������� ����� ��� ��� ����� ���� �� ������� � ������.
��� ����� ������������ ��������� `Type &refer = ....;`, ������� ����� ����� ��������� ��������� �� ������. 
��� ����������� ������ �� ������� ����� ���� ����� ���������� - ��������� ���������� ������� ��� ���������� �������, 
��������� dangling reference - ������ �� ������� ������. ��������� �� ��� - UB. [stackowerflow](https://stackoverflow.com/questions/46011510/what-is-a-dangling-reference)


```c++
void foo(std::vector<int> a) {
    a.push_back(1);
    std::cout << a.size(); 
} 

void bar(std::vector<int>& a) {
    a.push_back(1);
    std::cout << a.size();                                  
} 


int main() {
    std::vector<int> a{0};
    foo(a); // 2
    std::cout << a.size(); // 1 
    bar(a); // 2
    std::cout << a.size(); // 2  
    return 0;
}
```

������ � dangling reference

```c++
#include <iostream>
#include <vector>

std::vector<int>& foo() {
    std::vector<int> vec{1, 2, 3};
    return vec;
}

int main() {
    std::vector<int> vec = foo();
    std::cout << vec.size() << "\n"; // UB
}
```

## Storage duration

Storage duration - �������������� ��������, ��������� �� ������� ����� - �������, ����� ��� ��������� � �������.
[cppreference](https://en.cppreference.com/w/cpp/language/storage_duration) - ������ ��������.
�� ������������� automatic, static � dynamic.

### Automatic storage duration

�������� ��������������� storage duration - �� ������ ������ ��� ������� ����� ���������� ���������� - � �������� 
(������ ��� ������� �� ����, �� �������� ��� ����� �� �����������). ������ �������, ����� ��� ���������� 
���������� ��������� �������� (shadowing �� ���������, �.�. � �����-�� ������ ���������� ����� ���������� ��������).
� ����� ��������� � automatic storage duration - ����� �� storage duration. 

```c++
int main() {
    std::vector<int> v;                    // (1) - created

    for (int i = 0; i < 10; i++) {
        std::vector<int> v;                // (2) - created (10 times)
        if (i % 2 == 0) {
             break;                        // (2) - deleted
        }
    }                                      // (2) - deleted
    v.push_back(1);                      
    return 0;                              // (1) deleted
}


```

### Static storage duration

���������� ���������������� � �����-�� ������ � ����� �� ��������� ���� ���������. �������� ������ -
���������� ����������. ����� ������� � ��������� ������� � ����� �� storage duration - ��� �����
�� ����� �������� `static Type var;`. ��� ��������������� ������ ��� ������ ����������� ����� ������� � ��
����������, � ����� ���������� �� ����� ���������� ���������. ��� �� ������������� �������� ���� ������� �������, 
������������ � ���� ������ (� ��������� - ��������� �������, � ������� ��� �������).

```c++
std::vector foo(1000, 0);                 // (1) - created before main

int count(int start) {
    static int current = start;           // (2) - Created on first call; 
    return current++;
}

int bad_counter(int b){
    static int a{};                       // (3) Value-initialzation on first call
    a = b;                                // assigment on every call
    return a++;                                
}

int main() {
    foo.push_back(0);
    std::cout << count(5) << std::endl;           // 5
    std::cout << count(5) << std::endl;           // 6
    std::cout << count(0) << std::endl;           // 7
    std::cout << count(101) << std::endl;         // 8
    std::cout << bad_counter(5) << std::endl;     // 5
    std::cout << bad_counter(5) << std::endl;     // 5
    std::cout << bad_counter(0) << std::endl;     // 0
    std::cout << bad_counter(101) << std::endl;   // 101
}

```

������ ����� ���������� ���������� � ������� ���������� ����������, ������� �� �������� ����� ����� ���������
������� ������� ������ ����� ��������, ����� �� ������� ������ �� �����.

����� ����� ��������� ��� ��������, ��������� � SIOF [����� 33](https://github.com/khbminus/CppTickets/blob/master/tickets/ticket33.md).
��� �������� (��� ��� ��������� ������������� ����� ���������� ����� �������� [���](https://en.cppreference.com/w/cpp/language/initialization#Non-local_variables))
� � ������ ��� �������������


### Dynamic storage duration

��������� ���� ��������� ��������� �������� �����. ��� ������ ��������� `new Type;` - ��������
������, �������� ����� ��������� - ��������� �� ����. ��� ����, ������ ���������� ������ ������������
`delete ptr;`.

��� ������� ������� ������ 2 ���� ��� ������� ������, �������� �� ��� ������ ��������� `new` - 
��������� UB.
              
```c++

struct Foo {
    std::vector<int> vec(100);
}

int main() {
    Foo *ptr = new Foo;
    int bar;
    std::cout << ptr->vec.size() << std::cout;
    delete ptr; // To avoid memory leak
    //delete ptr; // Double free - UB; 
    //delete &bar; // UB;
}

```

�������������� ������ ���� �� ��������� ��������� - ������ ����������� �� � �����������. ����� �������� ���������� ������� ������, 
��� ����� ������� ���������� ��������.

���� ����� ������� new/delete; 

����� �������� ����� ������� ��� ������ `new Type[n]` - � ����� ������ ����������� ������ ������� ��� ������ `delete[] ptr` - 
����� UB (� ��� ����� ��� ������� ������� `delete ptr;`). 

��� new �������� ����� �������������:

* default - `new Type`
* default - `new Type()`
* default - `new Type{}`
* direct - `new Type(10)`
* direct list - `new Type{10}`

* ������������� �������� - `new Foo[n]{val1, val2, val3}` - �������������������� ���������������� �� ���������.




## ����� ����� ��������� ��������

��������� ������� ������� �� ���������� ���������� ��������� ��� ��� ��������, �� ��� ���� ���� �������
����������� ������ �� ��������� ������, ��� ����� ����� ���������, ����� �������������� ������� ����� ���� ������.

�����, ��� ������ ��������, ���� �� �������������� ����� ������ �� ��������� ������ ������.


```c++
int main() {
    const std::vector<int> &first = std::vectorP{0, 0, 0};
    std::cout << first.size() << std::endl; // NO UB
    const std::vector<int> &second = std::vectorP{0, 0, 0}; // would have been dangling if lifetime of second had been wider than lifetime of first;     
}
```

��� ����� ����� ��������, ���� 


```c++
int const& func(int const& x) {
    return x;
} 


int main() {
    const std::vector<int> &first = func(1);
    std::cout << first; // UB
}
```
�.�. ����� ����� ������� ������������ ������ �� ������� ����� x. x - �������� ����� ���������� ���������� �������.

�������� ����� ��������� � � STL - ������� std::min - ��������� � ���������� ������

```c++
int main() {
    const auto &val = std::min(0, 1); // Dangling reference
}
```
[�� ��� �������� range-based for ��������� � ���, ��� ��� �������������� - ��������� ���������](https://github.com/Nekrolm/ubbook/blob/master/lifetime/for_loop.md) 
