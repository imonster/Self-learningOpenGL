# 你好，三角形
* 顶点数组对象（VAO）: Vertex Array Object
* 顶点缓冲对象（VBO）: Vertex Buffer Object
* 索引缓冲对象（EBO、IBO）: Element Buffer Object、Index Buffer Object

## 图形渲染管线
可以被划分为两个主要部分：
1. 3D坐标转换为2D坐标
2. 2D坐标转变为实际有颜色的像素

可以被划分为几个阶段：
1. 顶点着色器
2. 形状（图元）装配
3. 几何着色器
4. 光栅化
5. 片段着色器
6. 测试与混合

##### 顶点着色器
它把一个单独的顶点作为输入
主要目的是把3D坐标转换为另一种3D坐标，同时对顶点属性进行一些基本处理

##### 图元装配
将顶点着色器输出的所有顶点作为输入
所有的点装配成指定图元形状

##### 几何着色器
把图元形式的一系列顶点的集合作为输入
可以通过产生新顶点构造出新的图元来生成其他形状

##### 光栅化、裁切
几何着色器的输出作为输入
把图元映射为最终屏幕上相应的像素，生成供片段着色器使用的片段
在片段着色器运行之前会执行裁切，丢弃超出视图以外的所有像素，提升执行效率

##### 片段着色器
光栅化生成的片段作为输入
主要目的是计算一个像素的最终颜色

##### 测试与混合
片段着色器的输出作为输入
检测片段的对应的深度值，判断这个像素是其它物体的前面还是后面，决定是否应该丢弃；检查Alpha值并对物体进行混合

## 数据输入
定义一个`float`数组：
```
//以标准化设备坐标形式
float vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
};
```
定义这样的顶点数据以后，把它作为输入发送给图形渲染管线的第一个处理阶段：顶点着色器

*在GPU上创建内存用于储存顶点数据，还要配置OpenGL如何解释这些内存，并且指定其如何发送给显卡*

通过VBO管理这个内存

使用`glGenBuffers`函数和缓冲ID生成一个缓冲对象：
```
unsigned int VBO;
glGenBuffers(1, &VBO);
```

OpenGL有很多缓冲对象类型，VBO的缓冲类型是`GL_ARRAY_BUFFER`
使用`glBindBuffer`函数指定缓冲对象类型是顶点缓冲对象
```
glBindBuffer(GL_ARRAY_BUFFER, VBO);
```

然后使用`glBufferData`函数，把定义的顶点数据复制到缓冲内存中
```
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

## 着色器
1）编写程序

```
//顶点着色器程序
#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}

//片段着色器程序
#version 330 core
out vec4 FragColor;

void main()
{
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
} 
```

2）编译源码
使用`glCreateShader`创建一个着色器对象
```
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);
```
`GL_VERTEX_SHADER`表示顶点着色器类型
`GL_FRAGMENT_SHADER`表示片段着色器类型

把着色器源码附加到着色器对象上，然后编译它：
```
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
glCompileShader(vertexShader);
```

检测编译是否成功：
```
int  success;
char infoLog[512];
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);

if(!success)
{
    glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
}
```

3）链接着色器程序
使用`glCreateProgram`创建一个程序对象
```
unsigned int shaderProgram;
shaderProgram = glCreateProgram();
```

把编译的着色器对象附加到程序上，然后链接它们：
```
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
glLinkProgram(shaderProgram);
```

检测链接是否成功：
```
int  success;
char infoLog[512];
glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);

if(!success)
{
    glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
    std::cout << "ERROR::SHADER::VERTEX::LINK_FAILED\n" << infoLog << std::endl;
}
```

4）激活、使用程序
使用`glUseProgram`函数激活着色器程序对象

把着色器对象链接到程序对象以后，记得删除着色器对象：
```
glDeleteShader(vertexShader);
glDeleteShader(fragmentShader);
```

*我们已经把输入顶点数据发送给了GPU，并指示了GPU如何在顶点和片段着色器中处理它。但是，OpenGL还不知道它该如何解释内存中的顶点数据，以及它该如何将顶点数据链接到顶点着色器的属性上。*

## 链接顶点属性
使用`glVertexAttribPointer`函数解析顶点数据；
使用`glEnableVertexAttribArray`启动顶点属性，默认是禁用的；
```
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);
```