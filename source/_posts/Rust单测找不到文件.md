---
title: Rust单测找不到文件
date: 2022-10-08 23:42:43
tags: ['Rust', '单元测试']
---

初学Rust, 在编写测试的时候看[官方文档](https://doc.rust-lang.org/rust-by-example/testing/unit_testing.html)中说只需要写一个mod如下即可：
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_sqrt() -> Result<(), String> {
        let x = 4.0;
        assert_eq!(x, 4.0);
        Ok(())
    }
}
```

而我在照做然后运行`cargo test`的时候却出现了`running 0 tests`的结果，搜索了一下发现原来是需要在`main.rs`中声明使用刚才定义的mod，[解决方法](https://stackoverflow.com/questions/44873527/cargo-not-running-tests-in-top-level-file)是正在`main.rs`中加入`mod tests`即可。

## 附加内容: 引用其他路径下的文件
一般来说我们的单测会放在一个文件夹中，这里拿文件夹`test`举例。这个时候想要引用到文件夹内，或者说不是跟`main.rs`同一目录的文件时，我们需要给我们原本的mod语句加上一个宏来制定文件的路径，例如：
```rust
#[path ="./test/your_test_file.rs"]
mod your_test_mod;
```

## TODOS