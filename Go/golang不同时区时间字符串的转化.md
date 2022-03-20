# golang不同时区时间字符串的转化

golang不同时区时间字符串的转化

使用场景：
 已知一个字符串时间"2020-01-30 15:30:30", 转化成不同的时区时间。
 假设上述字符串是CST时间，那么如何知道此时的UTC时间对应的字符串呢，以及反过来的需求。

这里实现两个golang函数，把时间字符串和时区转化成UTC时间字符串，和把UTC字符串转化成给定的时区字符串：

```go
package main

import (
    "fmt"
    "time"
)

const LOGTIMEFORMAT = "2006-01-02 15:04:05"

func utcToZone(t string, zone string) (string, error) {
    d, err := time.Parse(LOGTIMEFORMAT, t)
    if err != nil {
        return "", err
    }

  //loc, err := time.LoadLocation("Local")
    loc, err := time.LoadLocation(zone)
    if err != nil {
        return "", err
    }

    d = d.In(loc)
    return d.Format(LOGTIMEFORMAT), nil
}

func zoneToUTC(t string, zone string) (string, error) {
    d, err := time.Parse(LOGTIMEFORMAT + " MST", t + " " + zone)
    if err != nil {
        return "", err
    }

    loc, err := time.LoadLocation("UTC")
    if err != nil {
        return "", err
    }

    d = d.In(loc)
    return d.Format(LOGTIMEFORMAT), nil
}

func main() {
    in := "2020-01-30 15:30:30"
    if out, err := zoneToUTC(in, "CST"); err != nil {
        fmt.Printf("ERROR: %v\n", err)
    } else {
        fmt.Printf("%s CST == %s UTC\n", in, out)
    }

    if out, err := utcToZone(in, "EST"); err != nil {
        fmt.Printf("ERROR: %v\n", err)
    } else {
        fmt.Printf("%s UTC == %s EST\n", in, out)
    }
}
```

运行结果如下：

```undefined
2020-01-30 15:30:30 CST == 2020-01-30 07:30:30 UTC
2020-01-30 15:30:30 UTC == 2020-01-30 10:30:30 EST
```

意思是：

1. CST时间2020-01-30 15:30:30对应的UTC时间是2020-01-30 07:30:30 (中国时间是GMT+8)
2. UTC时间2020-01-30 15:30:30对应的EST时间是2020-01-30 10:30:30 (美东时间是GMT-5)