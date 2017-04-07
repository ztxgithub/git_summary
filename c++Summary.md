# c++用法

- 字符串的截取

``` c++

	istringstream parse(str);
	string token;

	for(int i = 0; getline(parse, token, ',') && i < this->cell_number; i++ ) {
        this->cell_capacity[i] = (uint32_t) atol(token.c_str());
    }
	
	截取完成条件：当parse都截取完了，parse内容会改变，token每次调用会被替换
	
```
