# FSU设备组编程规范

- 如果是strct 结构体用　下标_t

``` c
    
    struct adpt_ssbi_t
    {
        UINT8_T type;             ///< The signal point's type, AI, AO, DI, DO.
        UINT8_T dev_type;       ///< The device's type of the signal point belongs to.
        int addr;               ///< Relative offset of the register.
        int addr_len;           ///< The register's length.
        unsigned int bit_shift:4;        ///< If the signal point's type is DI,
        int scale;              ///< Decimal places of the point's value.
        union sig_val_t val;    ///< The signal point's value.
        char name[NAME_SIZE];   ///< The signal point's name.
        unsigned int valid:1;          ///< Whether this point is valid.
    };
        
        
```

- 指针 *　放在离变量名近的地方

``` c
    
    static UINT32_T to_uint32(UCHAR_T *value, int byte_num);
           
```

- 如果函数内需要的数组大小超过128字节,则需要动态申请内存

``` c
    
    如果 num <= 128
    char name[num];
    
    如果　num > 128
    char *name = (char *)malloc(sizeof(char));
   
```

- 函数名和变量名命名时尽量简洁.

- 如果是 enum 枚举类型 跟函数名一样

``` c
    
enum packet_type_c1 {
    LOGIN_C1 = 101,
    LOGIN_ACK_C1 = 102,
};
   
```
