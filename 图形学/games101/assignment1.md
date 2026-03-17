> [!NOTE]
>
> **完成MVP变换**

------

#### **三角形类**

```c++
class Triangle
{
  public:
    //逆时针存储的三角形三个点的坐标
    Vector3f v[3]; /*the original coordinates of the triangle, v0, v1, v2 in
                      counter clockwise order*/
    /*Per vertex values*/
    //三角形每个点的颜色，纹理坐标，法线
    Vector3f color[3];      // color at each vertex;
    Vector2f tex_coords[3]; // texture u,v
    Vector3f normal[3];     // normal vector for each vertex

    // Texture *tex;
    //三角形初始化
    Triangle();

    //用a,b,c的方式来取三角形中的三个点坐标
    Vector3f a() const { return v[0]; }
    Vector3f b() const { return v[1]; }
    Vector3f c() const { return v[2]; }

    void setVertex(int ind, Vector3f ver); /*set i-th vertex coordinates */
    void setNormal(int ind, Vector3f n);   /*set i-th vertex normal vector*/
    void setColor(int ind, float r, float g, float b); /*set i-th vertex color*/
    void setTexCoord(int ind, float s,
                     float t); /*set i-th vertex texture coordinate*/
    std::array<Vector4f, 3> toVector4() const;
};
```

成员函数

```c++
//构造函数，将三个点的坐标，颜色，纹理坐标都初始化为全0
Triangle::Triangle()
{
    v[0] << 0, 0, 0;
    v[1] << 0, 0, 0;
    v[2] << 0, 0, 0;

    color[0] << 0.0, 0.0, 0.0;
    color[1] << 0.0, 0.0, 0.0;
    color[2] << 0.0, 0.0, 0.0;

    tex_coords[0] << 0.0, 0.0;
    tex_coords[1] << 0.0, 0.0;
    tex_coords[2] << 0.0, 0.0;
}

//ind是点的序号，ver是坐标，该函数将三角形的三个坐标填满
void Triangle::setVertex(int ind, Eigen::Vector3f ver) { v[ind] = ver; }
//将三角形的三个法线填满
void Triangle::setNormal(int ind, Vector3f n) { normal[ind] = n; }
//将三角形的三个颜色填满（附加了RGB的范围检测和归一化）
void Triangle::setColor(int ind, float r, float g, float b)
{
	//rgb数据的范围检测，确保在0-255之间
    if ((r < 0.0) || (r > 255.) || (g < 0.0) || (g > 255.) || (b < 0.0) ||
        (b > 255.)) {
        throw std::runtime_error("Invalid color values");
    }

	//rgb归一化，因为大多数图形库使用0-1之间的浮点数表示颜色
    color[ind] = Vector3f((float)r / 255., (float)g / 255., (float)b / 255.);
    return;
}
//将三角形的纹理坐标填满
void Triangle::setTexCoord(int ind, float s, float t)
{
    tex_coords[ind] = Vector2f(s, t);
}
//常成员函数，无法修改成员变量的值。将三维坐标转化为齐次坐标
std::array<Vector4f, 3> Triangle::toVector4() const
{
    std::array<Vector4f, 3> res;
    std::transform(std::begin(v), std::end(v), res.begin(), [](auto& vec) {
        return Vector4f(vec.x(), vec.y(), vec.z(), 1.f);
    });
    return res;
}

```

------

#### 自己写的代码

```c++
#include "Triangle.hpp"
#include "rasterizer.hpp"
#include <eigen3/Eigen/Eigen>
#include <iostream>
#include <opencv2/opencv.hpp>

constexpr double MY_PI = 3.141592654;

//这里只考虑了旋转
Eigen::Matrix4f get_model_matrix(float rotation)
{
	Eigen::Matrix4f model_matrix = Eigen::Matrix4f::Identity();
	model_matrix << cos(rotation), -sin(rotation), 0, 0,
		sin(rotation), cos(rotation), 0, 0,
		0, 0, 1, 0,
		0, 0, 0, 1;
	return model_matrix;
}

//Eigen::Matrix4f get_view_matrix(float t, float g, float e)
//这里的相机位姿是正的，只需要平移
Eigen::Matrix4f get_view_matrix(Eigen::Vector3f e)
{
	Eigen::Matrix4f view_matrix = Eigen::Matrix4f::Identity();
	view_matrix << 1, 0, 0, -e[0],
		0, 1, 0, -e[1],
		0, 0, 1, -e[2],
		0, 0, 0, 1;
	return view_matrix;
}

Eigen::Matrix4f get_projection_matrix(float fov, float aspect, float near, float far)
{
	float half_fov = fov * 0.5 * MY_PI / 180;
	float top = near * tan(half_fov);
	float right = top * aspect;
	float bottom = -top;
	float left = -right;
	Eigen::Matrix4f pers2ortho;
	Eigen::Matrix4f ortho_matrix;
	pers2ortho << near, 0, 0, 0,
		0, near, 0, 0,
		0, 0, near + far, -near * far,
		0, 0, 1, 0;
	ortho_matrix << 2 / (right - left), 0, 0, -(right + left) / (right - left),
		0, 2 / (top - bottom), 0, -(top + bottom) / (top - bottom),
		0, 0, 2 / (near - far), -(near + far) / (near - far),
		0, 0, 0, 1;
	Eigen::Matrix4f projection_matrix;
	projection_matrix = ortho_matrix * pers2ortho;
	return projection_matrix;
}

int main()
{
	float angle = 0.0f;
	rst::rasterizer r(700, 700);
	Eigen::Vector3f eyePos = { 0.0f, 0.0f, 5.0f };

	std::vector<Eigen::Vector3f> pos{ {2.0f, 0.0f, -2.0f}, {0.0f, 2.0f, -2.0f}, {-2.0f, 0.0f, -2.0f} };
	std::vector<Eigen::Vector3i> ind{ {0, 1, 2} };

	auto posHandle = r.load_positions(pos);
	auto indHandle = r.load_indices(ind);

	int key = 0;
	int frameCount = 0;

	while (key != 27)
	{
		r.clear(rst::Buffers::Color | rst::Buffers::Depth);

		//get_model_matrix和get_rotation都是输出的M的位置，所以是二选一的
		r.set_model(get_model_matrix(angle));
		//r.set_model(get_rotation(angle, { 1.0f, 1.0f, 1.0f }));
		r.set_view(get_view_matrix(eyePos));
		r.set_projection(get_projection_matrix(45.0f, 1.0f, 0.1f, 50.0f));

		r.draw(posHandle, indHandle, rst::Primitive::Triangle);

		cv::Mat image(700, 700, CV_32FC3, r.frame_buffer().data());
		image.convertTo(image, CV_8UC3, 1.0f);
		cv::imshow("image", image);

		angle -= 0.05f;
		std::cout << "frame count: " << frameCount++ << '\n';

		key = cv::waitKey(10);
	}

	return 0;

}
```

##### `get_projection_matrix`函数

**输入参数**：近平面near，远平面far，垂直视野角fov，宽高比aspect

- 利用fov和near可以算出top，再通过aspect便可算出right。left和bottom取负即可
