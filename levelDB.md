# levelDB 使用

## 注意事项

```c++
1. leveldb::Slice 使用时，Sliece 是要依赖于外部的字符串数组，
  下面是错误代码:
leveldb::Slice slice;
if (...) {
  std::string str = ...;
  slice = str;
}
Use(slice);  // 这边 slice 依赖的 str 因为作用域的影响而被析构了
```

## 使用

```c++
1. leveldb::Iterator* it = db->NewIterator(options);
for (it->SeekToFirst(); it->Valid(); it->Next()) {
  ...
}

    2. it->key().starts_with(prefix)  // 判断是否为 prefix 开头
```

## 编译生成动态链接库
```shell
    > mkidr build && cd build
    > cmake -DBUILD_SHARED_LIBS=ON ..
    > make
```