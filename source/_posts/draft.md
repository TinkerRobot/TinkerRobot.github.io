## 方法

### 定义和调用方法

{% mahgen 123z456m789p %}

```rust
// 定义方法
impl Rectangle {
    fn area(&self) -> u32 {
        return self.width * self.height;
    }

    fn double(&mut self) {
        self.width *= 2;
        self.height *= 2;
    }
}

// ...

let mut rect = Rectangle {
    width: 30,
    height: 50,
};

// 调用方法
rect.double();
let area = rect.area();  // 6000
```

### 带参数的方法

```rust
// 定义方法
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        return self.width > other.width && self.height > other.height;
    }
}

// ...

let r1 = Rectangle {
    width: 10,
    height: 20,
};

let r2 = Rectangle {
    width: 5,
    height: 10,
};

// 调用方法
let can_hold = r1.can_hold(&r2);  // true
```

### 关联函数

```rust
// 定义关联方法
impl Rectangle {
    fn new(width: u32, height: u32) -> Self {
        return Rectangle {
            width,
            height,
        };
    }
}

// 调用关联方法
let rect = Rectangle::new(10, 20);
```