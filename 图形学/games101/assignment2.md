> [!NOTE]
>
> 完成rasterizer中的`insideTriangle`和`rasterizer_triangle`两个函数

------

#### 1.对`computeBarycentric2D`的修改

原代码采用了行列式法计算有向面积，本质上也是用面积比来计算重心坐标。但是原代码`insideTriangle`功能多余了，用重心坐标是否全大于零就可以判断是否在三角形内

```c++
double computeBarycentric2D(float ax, float ay, float bx, float by, float cx, float cy)
{
    return .5 * ((by - ay) * (bx + ax) + (cy - by) * (cx + bx) + (ay - cy) * (ax + cx));
}
```

------

#### 2.`rasterize_triangle`

- 函数输入：一个已经经过了MVP变换与视口变换的三角形
- 函数返回值：无
- 函数功能：遍历屏幕的所有像素，通过z-buffer判断三角形的遮挡情况，调用set_pixel进行逐像素渲染

```c++
void rst::rasterizer::rasterize_triangle(const Triangle& t)
{
    //t是一个有三个元素的数组，每个元素都是一个三维向量，表示三角形的三个顶点的坐标
    //tovector4()函数将每个顶点的三维坐标转换为四维坐标，添加了一个齐次坐标w=1.0
    //这里t中存储的已经是经过MVP和视口变换的屏幕坐标
    auto v = t.toVector4();

    auto [ax, ay, bx, by, cx, cy] = std::tuple{ v[0].x(), v[0].y(), v[1].x(), v[1].y(), v[2].x(),v[2].y() };

    //由于bounding box是以像素为单位，所以一定是整数
    int xmin = std::floor(std::min(ax, std::min(bx, cx)));
    int xmax = std::floor(std::max(ax, std::max(bx, cx)));
    int ymin = std::floor(std::min(ay, std::min(by, cy)));
    int ymax = std::floor(std::max(ay, std::max(by, cy)));

    for (int x = xmin; x <= xmax; x++)
    {
        for (int y = ymin; y <= ymax; y++)
        {
            double S_abc = computeBarycentric2D(ax, ay, bx, by, cx, cy);
            double S_pab = computeBarycentric2D(x, y, ax, ay, bx, by);
            double S_pbc = computeBarycentric2D(x, y, bx, by, cx, cy);
            double S_pca = computeBarycentric2D(x, y, cx, cy, ax, ay);
            float alpha = S_pbc / S_abc;
            float beta = S_pca / S_abc;
            float gamma = S_pab / S_abc;
            if (insideTriangle(alpha, beta, gamma))
            {
                float insert_depth = alpha * v[0].z() + beta * v[1].z() + gamma * v[2].z();
                //这里三角形的三个点颜色都是一样的，所以不需要搞插值
                Eigen::Vector3f color = t.getColor();
                //z-buffer
                if (insert_depth < depth_buf[get_index(x, y)])
                {
                    depth_buf[get_index(x, y)] = insert_depth;
                }
                Eigen::Vector3f p = { float(x), float(y), insert_depth };
                //渲染像素
                set_pixel(p, color);
            }
        }
    }
}
```

1. 计算bounding box
2. 遍历bounding box中的所有像素
   1. 计算该像素的重心坐标，并利用重心坐标判断该像素是否在三角形内
   2. 若在三角形内，利用重心坐标计算颜色与深度插值，通过z-buffer判断是否被遮挡
   3. 若不被遮挡，将该像素设置为对应颜色