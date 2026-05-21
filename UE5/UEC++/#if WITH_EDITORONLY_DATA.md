```c++
#if WITH_EDITORONLY_DATA
	/** Custom serialization */
	UE_API void PostSerialize(const FArchive& Ar);
#endif
```

`#if#endif`是预处理指令，在预处理的时候做判断

`WITH_EDITORONLY_DATA`如果被启用，则中间的代码只会给编辑器使用，打包游戏时会被移除

