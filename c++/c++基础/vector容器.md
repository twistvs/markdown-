`data()`  返回vector中第一个元素的指针

```c++
//frame_buffer是vector容器
//返回frame_buffer的第一个元素的指针
cv::Mat image(700, 700, CV_32FC3, r.frame_buffer().data());
```

