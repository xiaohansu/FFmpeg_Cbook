### AVDictionary

[TOC]

AVDictionary是一个健值对存储工具，类似于c++中的map，ffmpeg中有很多 API 通过它来传递参数。





## code

libavutil/dict.h

```c
typedef struct AVDictionaryEntry {  
    char *key;  
    char *value;  
} AVDictionaryEntry;  

struct AVDictionary {  
    int count;  
    AVDictionaryEntry *elems;  
};
```

## demo

## link

https://www.jianshu.com/p/89f2da631e16?utm_source=wechat_session