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


- switch-case,case下面一定要加{}

``` c++

      case FAST: // 冲放电事件情况下
            {
                fastSampleSend();
                loopSampleData();
                bool isEvent = BatMgrCheckAlarmAndEvent();
                if (isEvent) {
                    changeState(FAST);
                } else if (this->sampleState == FAST) {
                    changeState(SLOW);
                }
            }
	
```

- 类型转化要进行强制转换

``` c++

	int64_t cap = this->temp_used_capacity;
	int32_t current = data.getCurrent();
	uint32_t diff_in_ms = timeDiffInMS(&this->pre_time, current_time);
	cap += ((int64_t)current * (int64_t)diff_in_ms);
	
	注意：如果没有强制转换：cap += (current * diff_in_ms);会溢出
	
```

- list::erase

``` c++

	iterator erase (const_iterator position);
	
	iterator erase (const_iterator first, const_iterator last);
	删除的范围是 [first,last)，不包括last
	
	返回值：指向下一个可用的iterator
	
```

- printf输出格式

``` c++
	uint32_t --> %u
	int32_t  --> %d
	
```