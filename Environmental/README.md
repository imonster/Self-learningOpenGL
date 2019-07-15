# Environmental Construction
OpenGL环境搭建，创建HelloWorld工程

* GLFW
* GLAD
* OpenGL 3.3

### 构建GLFW
GLFW是一个专门针对OpenGL的C语言库，它提供了一些渲染物体所需的最低限度的接口。它允许用户创建OpenGL上下文，定义窗口参数以及处理用户输入。

[下载](https://www.glfw.org/download.html)源代码包
![屏幕快照](https://i.loli.net/2019/07/10/5d255bd300ba951722.png)
-------

编译源代码需要CMake工具

[下载](https://cmake.org/download/)安装CMake

启动CMake

在GLFW源代码的根目录中新建build文件夹，作为编译的目标目录

选择GLFW源代码根目录作为源代码目录，根目录中的build目录作为目标文件目录
![屏幕快照](https://i.loli.net/2019/07/10/5d2561088954464174.png)

点击按钮Configure，这时会有弹框
![屏幕快照](https://i.loli.net/2019/07/10/5d256205032df73565.png)

默认设置，直接点击Done

![屏幕快照](https://i.loli.net/2019/07/10/5d25653553d5436294.png)

再点击按钮Configure
![屏幕快照](https://i.loli.net/2019/07/10/5d25658fd6b7a84363.png)

点击按钮Generate，会在build目录中生成工程文件
![屏幕快照](https://i.loli.net/2019/07/10/5d25668080b2b92602.png)

打开终端，进入build目录，执行`make`命令，进行编译安装库文件
![屏幕快照](https://i.loli.net/2019/07/10/5d25676ab7c8c64490.png)

完成后，执行`make install`命令进行安装，一般会安装到目录`/usr/local/include`和`/usr/local/lib`中
如果在执行命令时出现权限错误时，可以执行命令`sudo make install`
![屏幕快照](https://i.loli.net/2019/07/10/5d2568cd9eda936538.png)

安装成功显示
![屏幕快照](https://i.loli.net/2019/07/10/5d25693ede23f16062.png)

至此，macOS下GLFW环境配置完成！

### 配置GLAD
GLAD是用来管理OpenGL的函数指针

* 打开[在线服务](https://glad.dav1d.de)
* Language选择C/C++
* Specification选择OpenGL
* API选择Version 3.3
* Profile选择Core模式
* 忽略Extensions
* Options中选中Generate a loader

![屏幕快照](https://i.loli.net/2019/07/10/5d2575296261f70554.png)

点击GENERATE按钮

![屏幕快照](https://i.loli.net/2019/07/10/5d2576a916c7414993.png)

下载zip包，解压后将include目录下的glad和KHR文件夹复制到目录`/usr/local/include`，**其中src目录下的`glad.c`文件需要添加到项目中**。

### 新建Xcode项目
* 选择macOS下的Cocoa App
![屏幕快照](https://i.loli.net/2019/07/10/5d257cb84d51f79390.png)

生成默认的工程结构
![屏幕快照](https://i.loli.net/2019/07/10/5d258175c846831589.png)

这里需要删除一些用不到的文件，像`.h`、`.m`、`Main.storyboard`文件

* 配置环境
添加头文件搜索路径`/usr/local/include`和库文件搜索路径`/usr/local/lib`：
![屏幕快照](https://i.loli.net/2019/07/10/5d2582dd32b3440831.png)

添加相关库文件：
![屏幕快照](https://i.loli.net/2019/07/10/5d2587201312b97936.png)
其中，libglfw3.a就是在配置GLFW环境时生成的，需要去`/usr/local/lib`目录中添加，最后别忘了添加`glad.c`文件到项目中。

* 编写程序
新建`main.cpp` 文件

```
#include <glad/glad.h>
#include <GLFW/glfw3.h>

#include <iostream>

void framebuffer_size_callback(GLFWwindow* window, int width, int height);
void processInput(GLFWwindow *window);

// settings
const unsigned int SCR_WIDTH = 800;
const unsigned int SCR_HEIGHT = 600;

int main()
{
    // glfw: 初始化和配置
    // ------------------------------
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3); // 主版本号
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3); // 次版本号
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE); // 核心模式
    
#ifdef __APPLE__
    glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE); // macOS中需要加上这行代码，配置才能生效
#endif
    
    // glfw: 创建窗口对象
    // --------------------
    GLFWwindow* window = glfwCreateWindow(SCR_WIDTH, SCR_HEIGHT, "LearnOpenGL", NULL, NULL);
    if (window == NULL)
    {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    glfwMakeContextCurrent(window); // 将窗口的上下文设置为当前线程的主上下文
    glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
    
    // glad: 初始化，加载所有OpenGL函数指针
    // ---------------------------------------
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
    {
        std::cout << "Failed to initialize GLAD" << std::endl;
        return -1;
    }
    
    // 渲染循环
    // -----------
    while (!glfwWindowShouldClose(window))
    {
        // 输入
        // -----
        processInput(window);
        
        // 渲染
        // ------
        glClearColor(0.2f, 0.3f, 0.3f, 1.0f);
        glClear(GL_COLOR_BUFFER_BIT);
        
        // glfw: swap buffers and poll IO events (keys pressed/released, mouse moved etc.)
        // -------------------------------------------------------------------------------
        glfwSwapBuffers(window);
        glfwPollEvents();
    }
    
    // glfw: 正确释放/删除之前的分配的所有资源
    // ------------------------------------------------------------------
    glfwTerminate();
    return 0;
}

// process all input: query GLFW whether relevant keys are pressed/released this frame and react accordingly
// ---------------------------------------------------------------------------------------------------------
void processInput(GLFWwindow *window)
{
    if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)
        glfwSetWindowShouldClose(window, true);
}

// glfw: 在每次窗口大小被调整的时候被调用
// ---------------------------------------------------------------------------------------------
void framebuffer_size_callback(GLFWwindow* window, int width, int height)
{
    glViewport(0, 0, width, height);
}
```

### 运行
![屏幕快照](https://i.loli.net/2019/07/10/5d258df5879f388704.png)
至此，可以看见上图这样一个窗口。

## GitHub
[Self-learningOpenGL](https://github.com/imonster/Self-learningOpenGL)