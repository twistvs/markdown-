1. 读取obj文件获得：所有顶点的坐标，法线，纹理坐标，颜色；三角形的连接情况

2. 顶点的MVP+视口变换，得到所有屏幕坐标

3. 逐三角形进行光栅化：对三角形使用bounding box，然后遍历bounding box中的像素：重心坐标更新z-buffer，计算三角形中像素的颜色，法线，纹理坐标，深度插值；==利用提前存储好的view空间坐标，插值出该像素在空间中的坐标，用于计算着色==。==在有着色器的情况下==根据着色器重新计算像素的颜色，而不是直接使用颜色插值。

#### phong_fragment_shader着色器

```c++
Eigen::Vector3f phong_fragment_shader(const fragment_shader_payload& payload)
{
	//环境光系数ka，漫反射系数kd（直接用归一化颜色表示），高光系数ks
    Eigen::Vector3f ka{ 0.005f, 0.005f, 0.005f };
    Eigen::Vector3f kd = payload.color;
    Eigen::Vector3f ks{ 0.7937f, 0.7937f, 0.7937f };

	// 结构体light{position,intensity}
	//创建两个光源，光的强度也是一个三维的向量
    std::vector<light> lights = {
        light{{ 20.0f, 20.0f, 20.0f }, { 500.0f, 500.0f, 500.0f }},
        light{{ -20.0f, 20.0f, 0.0f }, { 500.0f, 500.0f, 500.0f }} };

    //环境光强度
    Eigen::Vector3f amb_light_intensity{ 5.0f, 5.0f, 5.0f };
	//观察者位置
    Eigen::Vector3f eye_pos{ 0.0f, 0.0f, 10.0f };

    constexpr float p = 150.0f;

	//法线，相机坐标系位置，视线方向，法线和视线方向是方向向量需要归一化
    Eigen::Vector3f normal = payload.normal.normalized();
    Eigen::Vector3f point = payload.view_pos;
    Eigen::Vector3f viewDir = (eye_pos - point).normalized();

    Eigen::Vector3f color{ 0.0f, 0.0f, 0.0f };

    //计算该像素在高光，漫反射，环境光下的最终颜色值
    for (auto& light : lights)
    {
        //点光源与点的差值向量
        Eigen::Vector3f lightDir = light.position - point;
        //利用该差值向量点乘自己，算出两点距离的平方
        float distance2 = lightDir.dot(lightDir);
        lightDir.normalize();
        //通过观察方向向量和光方向向量，可以计算出两者中间的半程向量h，半程向量也是方向向量
        Eigen::Vector3f halfDir = (viewDir + lightDir).normalized();
        //法线方向向量与光方向向量的点乘，用于计算漫反射
        float NdotL = normal.dot(lightDir);
        //法线方向向量与半程方向向量的点乘，用于计算高光反射
        float NdotH = normal.dot(halfDir);

        //提前把I除以r方，后面计算就不需要写r方了
        Eigen::Vector3f intensity = light.intensity / distance2;

        //环境光计算公式
        Eigen::Vector3f ambient = ka.cwiseProduct(amb_light_intensity);
        //漫反射计算公式
        Eigen::Vector3f diffuse = kd.cwiseProduct(intensity) * std::max(0.0f, NdotL);
        //高光计算公式
        Eigen::Vector3f specular = ks.cwiseProduct(intensity) * std::pow(std::max(0.0f, NdotH), p);
        //三个光加在一起
        color += (ambient + diffuse + specular);
    }

    //把归一化的颜色恢复到0-255之间
    return color * 255.0f;
}
```

![image-20260121111541122](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20260121111541122.png)

![image-20260121113213924](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20260121113213924.png)

![image-20260121113225264](C:\Users\lee\AppData\Roaming\Typora\typora-user-images\image-20260121113225264.png)