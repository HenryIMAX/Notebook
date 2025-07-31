# C++ 并行程序设计

## C++入门

### OOP（面向对象程序设计）

**Class（类）**：Data with Functions

Class的定义：

```cpp
class ClassName {
    public: // Public interface
    ClassName(); // Constructor
    ~ClassName(); // Destructor
    void publicMethod(); // Public method
    private: // Private implementation details
    int privateData; // Private data member
    void privateMethod(); // Private method
    protected: // Protected members (accessible by derived classes)
    int protectedData; // Protected data member
};
```

实例化:

```cpp
int main() {
    Point p(10, 20); // Create an object of Point
    p.display(); // Call public method
    return 0;
}
```

This Pointer:

```cpp
class Point {
    private:
    int x, y;
    public:
    Point(int x, int y) {
        this->x = x; // Use this to refer to member variable
        this->y = y; // Use this to refer to member variable
    }
};
```

