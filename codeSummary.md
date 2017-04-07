# 项目过程要用到的编码技巧

- 当要进行批量的转化一组数据时

``` c++

enum CurrentProbeRangeBDM {
    CURRENT_PROBE_RANGE_UNSET = 0,
    CURRENT_PROBE_RANGE_50A = 1,
    CURRENT_PROBE_RANGE_100A = 2,
    CURRENT_PROBE_RANGE_200A = 3,
    CURRENT_PROBE_RANGE_500A = 4,
};

BDMIter.currentProbeRange = BDM2currentProbeRange((CurrentProbeRangeBDM) range);

inline uint32_t BDM2currentProbeRange(CurrentProbeRangeBDM currentProbeRange) {
    switch (currentProbeRange) {
        case CURRENT_PROBE_RANGE_50A:
            return 50;
        case CURRENT_PROBE_RANGE_100A:
            return 100;
        case CURRENT_PROBE_RANGE_200A:
            return 200;
        case CURRENT_PROBE_RANGE_500A:
            return 500;
        default:
            log_w("bdm probeRange %d is UNKNOWN", currentProbeRange);
    }
    return 0;
}

```

- 创建ADT（抽象数据类型），将一个抽象具有的数据和操作放在一个文件内