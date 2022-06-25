# 简单的 Billboard 效果实现

## 介绍

Billboard 是游戏中一个比较常见的效果，其主要功能是让物体根据摄像机的位置进行旋转，使得玩家有一种该物体永远面向自己的感觉。传统的Billboard主要有两种，一是限定y轴的Billboard，二是自由旋转的Billboard，可用一个参数来实现切换。Billboard的实现方式也比较多，可以在应用阶段完成，即渲染前调整物体的模型矩阵使其朝向摄像机；也可以在Shader中完成。本篇文章主要介绍Billboard效果在Shader中的实现。

## 准备工作

首先，你需要准备一张面片作为Billboard的载体。注意面片需要直立放置，且正面朝前（平行于XY平面），中心位于（0,0,0）处，不同的放置方式会对后面的线性变换的方式产生影响。

<img src="https://github.com/Orznijiang/MyImageBed/blob/main/Code-Blog/CG/Effect/01.png?raw=true" alt="使用Blender制作的面片" style="zoom:50%;" />

若使用较为成熟的商业引擎，一种较为简单的方法是直接使用引擎内置的模型，如Unity的Quad，该模型在初始状态下就是直立放置。

<img src="https://github.com/Orznijiang/MyImageBed/blob/main/Code-Blog/CG/Effect/02.png?raw=true" alt="使用Unity创建内置Quad" style="zoom:80%;" />

![Unity内置Quad](https://github.com/Orznijiang/MyImageBed/blob/main/Code-Blog/CG/Effect/03.png?raw=true)

此外，还需要准备一张纹理，作为Billboard的贴图内容。



## 参数设计

| name                | initialization | explaining                              |
| ------------------- | -------------- | --------------------------------------- |
| tint_map            | white          | billboard纹理（支持blending）           |
| tint_color          | （1,1,1,1）    | billboard叠加颜色                       |
| vertical_restraints | 1.0            | 垂直约束（0：不限制；1：完全限制到y轴） |
| floating_phase      | 0.0            | 浮动的初始相位                          |
| floating_scale      | 1.0            | 浮动的幅度                              |
| floating_speed      | 1.0            | 浮动的速度                              |

其中，tint_map和tint_color共同决定Billboard的颜色。vertical_restraints如上所述，用于实现两种Billboard模式的切换。最后三个属性，floating_phase、floating_scale以及floating_speed属性，用来实现Billboard的缓动效果。



## 基本效果实现

在顶点着色器中，首先将摄像机的坐标转到Billboard面片的模型空间中：

```
vec3 viewer = (inverse(_MODEL_2_WORLD) * vec4(eye_pos, 1.0)).xyz;
```

我们设定Billboard面片的中心为原点位置，即（0,0,0）。这样，从面片中心到摄像机位置的向量的方向就可以看作是面片在变换后要看向的方向，也就是变换后的面片的实际法线方向：

```
vec3 center = vec3(0.0);
vec3 normal_dir = viewer - center;
```

接下来，要根据Billboard的两种实现类型，对normal方向进行修改。对于自由的Billboard，无需修改计算结果。对于限定y轴的Billboard，我们要清除normal的y分量，使normal方向永远垂直于y轴，最后对normal进行归一化：

```
normal_dir.y *= (1.0 - vertical_restraints);
normal_dir = normalize(normal_dir);
```

接下来是关键部分，我们要将normal作为一个基，并根据其计算出另外两个基：

```
vec3 up_dir = abs(normal_dir.y) > 0.999 ? vec3(0, 0, 1) : vec3(0, 1, 0);
vec3 right_dir = normalize(cross(up_dir, normal_dir));
up_dir = normalize(cross(normal_dir, right_dir));
```

先设定一个可用的up方向，再根据up和normal通过叉乘计算出right方向，然后使用right和normal方向对up方向进行修正。

![建立一个新的坐标系](https://github.com/Orznijiang/MyImageBed/blob/main/Code-Blog/CG/Effect/04.png?raw=true)

这里要注意的是，叉乘的顺序会对结果造成影响，上面是在右手坐标系下的叉乘顺序，对于左手坐标系，应交换叉乘顺序（叉乘永远都是右手定则），即：

```
vec3 right_dir = normalize(cross(normal_dir, up_dir));
up_dir = normalize(cross(right_dir, normal_dir));
```

否则，会造成意料之外的错误，如Billboard的背面永远朝向摄像机。在Cull Back的情况下，我们什么也看不到；在Cull Off的情况下，Billboard左右发生镜像。



运用这三个基对顶点进行线性变换，得到线性变换后的顶点位置：

```
vec3 center_offset = model_position.xyz - center;
vec3 local_pos = center + right_dir * center_offset.x + up_dir * center_offset.y + normal_dir * center_offset.z;
```

用矩阵表示，即为：
$$
\begin{bmatrix}
	right.x & up.x & normal.x \\
	right.y & up.y & normal.y \\
	right.z & up.z & normal.z \\
\end{bmatrix}
\begin{bmatrix}
	offset.x \\
	offset.y \\
	offset.z \\
\end{bmatrix}
$$
这里要注意offset的每个分量与每个基向量的对应关系，否则也会导致错误。实际上，可以忽视z分量的变换，因为我们的面片完全处于z=0的XY平面上。

使用变换后的顶点位置进行变换，输出裁剪空间坐标，同时输出uv信息：

```
uv = vertex_uv;
vec4 world_position = (_MODEL_2_WORLD * vec4(local_pos, 1.0));
gl_Position = _VIEW_2_CLIP * _WORLD_2_VIEW * world_position;
```

片元着色器的操作比较简单，就是对纹理进行采样并叠加上颜色，输出结果：

```
vec4 tex_color = texture(tint_texture, uv);
vec4 result = tex_color.rgba * tint_color.rgba;
fragment_color = result;
```

这样就完成了Billboard的基本功能，效果大致如下：

<img src="https://github.com/Orznijiang/MyImageBed/blob/main/Code-Blog/CG/Effect/05.png?raw=true" alt="未开启Blending的Billboard" style="zoom: 50%;" />

对于带有透明通道的纹理，可以开启Blending，以获得更好的效果：

```
pass.depth_stencil_state.should_enable_depth_testing = true;
pass.depth_stencil_state.should_write_depth = false;
pass.blend_state.should_enable_blending = true;
```

开启Blending的效果：

<img src="https://github.com/Orznijiang/MyImageBed/blob/main/Code-Blog/CG/Effect/06.png?raw=true" alt="开启Blending的Billboard" style="zoom:50%;" />



## 加入缓动效果

这里为Billboard添加一定的动效，使其在场景中缓慢地浮动。

glsl中没有直接获取时间信息的接口，需要传入uniform：

```
auto seconds = float(Window_System::current().seconds());
pass.set_uniform("time", seconds);
```

根据上面的参数设计，我们使用3个参数来控制周期性的浮动效果，分别影响初始位置、幅度以及速度：

```
vec3 up_floating = sin((time + floating_phase * pi) * floating_speed) * floating_scale * up_dir;
local_pos += up_floating;
```

这样，Billboard就能够在场景中进行周期性的缓慢运动。



## 踩坑

### 面片的制作

在DCC（Digital Content Creation）软件中创建的默认平面一般是平放状态，我们需要对其旋转90度使其默认为直立状态，否则会导致计算结果出现异常，包括：

* 面片完全不显示
* 面片不是看向摄像机，而是以自身原点的垂线为轴进行旋转

如果我们选择将旋转后的面片直接导出为FBX格式，则依旧会出现上面的问题，这可能是因为FBX保留了mesh的旋转信息，实际上面片在模型空间中还是平放的。对此，我们需要将面片的旋转操作应用到mesh的每个顶点上。在Blender中，对旋转后的物体进行下面的操作，或者在编辑模式下进行旋转：

<img src="https://github.com/Orznijiang/MyImageBed/blob/main/Code-Blog/CG/Effect/07.png?raw=true" alt="应用模型变换信息" style="zoom:80%;" />

若直接导出为OBJ格式的模型文件，测试没有上面的问题。



### 透明物体的渲染顺序

透明物体需要按大体从后往前的方式进行绘制，否则会产生混合计算错误的问题。如下，由于人物最先绘制，所以被后绘制的物体遮挡，即使深度上在人物后面（因为关闭了深度写入）：

![错误的渲染顺序](https://github.com/Orznijiang/MyImageBed/blob/main/Code-Blog/CG/Effect/08.png?raw=true)

忽视模型间可能出现的交叉现象，这里根据模型到摄像机的距离大致对渲染队列中的透明对象进行排序：

```
auto distance2 = [&](math::Vector3 from, math::Vector3 to) noexcept -> double {
	auto dis = math::Vector3(to - from);
	return dis.x * dis.x + dis.y * dis.y + dis.z * dis.z;
};

auto cmp = [&](ss::Draw_Command obj1, ss::Draw_Command obj2) noexcept -> bool {
	auto pos1 = scene::builtin_service::Transformation_Service::current().world_transformation(obj1.node).config().position;
	auto pos2 = scene::builtin_service::Transformation_Service::current().world_transformation(obj2.node).config().position;
	auto dis1 = distance2(pos1, camera_pos);
	auto dis2 = distance2(pos2, camera_pos);
	return dis1 > dis2;
};

std::sort(pass.queue.begin(), pass.queue.end(), cmp);
```

现在透明billboard的渲染就有了较为正确的遮挡关系：

<img src="https://github.com/Orznijiang/MyImageBed/blob/main/Code-Blog/CG/Effect/09.png?raw=true" alt="正确的渲染顺序" style="zoom: 50%;" />





## 参考文献

* https://zhuanlan.zhihu.com/p/29072964
* https://blog.csdn.net/weixin_43813453/article/details/101039775
* 《Unity Shader 入门精要》
* http://www.cppblog.com/SoRoMan/archive/2006/08/08/10978.html