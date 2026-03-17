```cmake
cmake_minimum_required(VERSION 3.10)
project(Rasterizer)

list(APPEND CMAKE_PREFIX_PATH "D:/yingyong/Visual Studio/tools/opencv4.12.0/opencv/build/x64/vc16/lib")

find_package(OpenCV REQUIRED)

set(CMAKE_CXX_STANDARD 17)

include_directories(D:/yingyong/Visual Studio/tools/opencv4.12.0/opencv/build/include)

add_executable(Rasterizer main.cpp rasterizer.hpp rasterizer.cpp Triangle.hpp Triangle.cpp)
target_link_libraries(Rasterizer ${OpenCV_LIBRARIES})
target_include_directories(Rasterizer PRIVATE "D:/yingyong/Visual Studio/tools/eigen-3.4.0/eigen-3.4.0")
```

`find_package`会在CMAKE_PRIFIX_PATH中寻找类似于OpenCVConfig.cmake的配置文件来一键配置库，但是在windows中，基本上不太可能直接找到（因为他找的是C盘的默认目录，而很多时候我们不会把文件放在C盘），所以可以通过自己把路径append到CMAKE_PRIFIX_PATH的方式来实现

eigen3不需要编译，可以直接用`target_include_directories`导进来（注意设置为PRIVATE模式）

