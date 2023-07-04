---
title: OpenGL环境配置
---
> 使用Glfw + Glad的方案，帮助你在本地配置OpenGL运行环境

## Glfw
https://www.glfw.org/

下载Glfw 3.3.8，选择32位，因为如果选择64位，将来编写的程序只能在64位机器上跑
![](image.png)

将include和lib-vc2019文件夹（方便起见，重命名为lib）拷贝到OpenGLlab项目（VS2019新建C++空白项目）工程目录下
![](image-1.png)

## Glad
https://glad.dav1d.de/
![](image-2.png)
选好红圈位置后点击生成
![](image-3.png)
![](image-4.png)
将include中的内容拷贝到OpenGLLab项目的include文件夹下，src目录的glad.c拷贝到项目源码目录下
![](image-5.png)
![](image-6.png)
将glad.c包含到项目中，下图是已在项目中的状态
![](image-7.png)
## 项目配置

![](image-8.png)
![](image-9.png)
## 测试是否配置成功
然后就可以愉快的使用OpenGL进行编程了，示例代码：
绘制四边形

``` c++
#include <iostream>
#include <glad/glad.h>
#include <GLFW/glfw3.h>

// 方法一
// 渲染之前VAO/VBO的绑定生成
// 顶点数据
const float vertices[] = {
	// 第一个三角形
	0.5f, 0.5f, 0.0f, // 右上
	0.5f, -0.5f, 0.0f, // 右下
	-0.5f, -0.5f, 0.0f, // 左下
	// 第二个三角形
	-0.5f, -0.5f, 0.0f, // 左下
	0.5f, 0.5f, 0.0f, // 右上
	-0.5f, 0.5f, 0.0f, // 左上
};

int screen_width = 1280;
int screen_height = 720;

int main(void)
{
	GLFWwindow* window;

	/* Initialize the library */
	if (!glfwInit())
		return -1;

	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
	glfwWindowHint(GLFW_OPENGL_ANY_PROFILE, GLFW_OPENGL_CORE_PROFILE);	// 使用核心模式，无需向后兼容
	glfwWindowHint(GLFW_RELEASE, false);

	/* Create a windowed mode window and its OpenGL context */
	window = glfwCreateWindow(screen_width, screen_height, "Quadrilateral", NULL, NULL);
	if (!window)
	{
		std::cout << "Failed to Create OpenGL Context" << std::endl;
		glfwTerminate();
		return -1;
	}

	/* Make the window's context current */
	glfwMakeContextCurrent(window);	// 将窗口的上下文设置为当前线程的主上下文

	// 初始化GLAD，加载OpenGL函数指针地址的函数
	if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
	{
		std::cout << "Failed to initialize GLAD" << std::endl;
	}

	// 指定当前视口尺寸（前两个参数为左下角位置，后两个参数为渲染窗口的宽和高）
	glViewport(0, 0, screen_width, screen_height);

	// 生成四边形的VAO、VBO
	unsigned int VBO, VAO;
	glGenVertexArrays(1, &VAO);
	glGenBuffers(1, &VBO);
	// 绑定VAO
	glBindVertexArray(VAO);
	// 绑定VBO并传入顶点数据
	glBindBuffer(GL_ARRAY_BUFFER, VBO);
	glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
	// 设置顶点属性指针
	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
	glEnableVertexAttribArray(0);
	// 解绑VBO
	glBindBuffer(GL_ARRAY_BUFFER, 0);
	// 解绑VAO
	glBindVertexArray(0);

	// 顶点着色器和片元着色器源码
	const char* vertex_shader_source =
		"#version 330 core\n"
		"layout (location=0) in vec3 aPos;\n"
		"void main()\n"
		"{\n"
		"	gl_Position=vec4(aPos, 1.0);\n"
		"}\n\0";
	const char* fragment_shader_source =
		"#version 330 core\n"
		"out vec4 FragColor;\n"
		"void main()\n"
		"{\n"
		"	FragColor=vec4(1.0f,0.5f,0.2f,1.0f);\n"// 三角形的颜色
		"}\n\0";

	// 生成并编译着色器
	// 顶点着色器
	int vertex_shader = glCreateShader(GL_VERTEX_SHADER);
	glShaderSource(vertex_shader, 1, &vertex_shader_source, NULL);
	glCompileShader(vertex_shader);
	int success;
	char info_log[512];
	// 检查着色器是否编译成功，如果编译失败，打印错误信息
	glGetShaderiv(vertex_shader, GL_COMPILE_STATUS, &success);
	if (!success)
	{
		glGetShaderInfoLog(vertex_shader, 512, nullptr, info_log);
		std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << info_log << std::endl;
	}

	// 片元着色器
	int fragment_shader = glCreateShader(GL_FRAGMENT_SHADER);
	glShaderSource(fragment_shader, 1, &fragment_shader_source, nullptr);
	glCompileShader(fragment_shader);
	// 检查着色器是否成功编译，如果编译失败，打印错误信息
	glGetShaderiv(fragment_shader, GL_COMPILE_STATUS, &success);
	if (!success)
	{
		glGetShaderInfoLog(fragment_shader, 512, nullptr, info_log);
		std::cout << "ERROR::SHADER::FRAGMENT::COMPILATION_FAILED\n" << info_log << std::endl;
	}
	
	// 链接顶点和片元着色器至一个着色器程序
	int shader_program = glCreateProgram();
	glAttachShader(shader_program, vertex_shader);
	glAttachShader(shader_program, fragment_shader);
	glLinkProgram(shader_program);
	// 检查着色器是否链接成功，如果链接失败，打印错误信息
	glGetProgramiv(shader_program, GL_LINK_STATUS, &success);
	if (!success)
	{
		glGetProgramInfoLog(shader_program, 512, nullptr, info_log);
		std::cout << "ERROR::SHADER::PROGRAM::LINKING_FAILED\n" << info_log << std::endl;
	}

	glDeleteShader(vertex_shader);
	glDeleteShader(fragment_shader);

	// 线框模式
	//glPolygonMode(GL_FRONT_AND_BACK, GL_LINE);
	
	/* Loop until the user closes the window */
	while (!glfwWindowShouldClose(window))
	{
		/* Render here */
		glClearColor(0.0f, 0.34f, 0.57f, 1.0f);	// 背景色
		glClear(GL_COLOR_BUFFER_BIT);

		// 使用着色器程序
		glUseProgram(shader_program);

		// 绘制三角形
		glBindVertexArray(VAO);
		glDrawArrays(GL_TRIANGLES, 0, 6);
		glBindVertexArray(0);

		/* Swap front and back buffers */
		glfwSwapBuffers(window);

		/* Poll for and process events */
		glfwPollEvents();
	}

	glDeleteVertexArrays(1, &VAO);
	glDeleteBuffers(1, &VBO);

	glfwTerminate();
	return 0;
}
```

如果没有问题你将会看到一个四边形
![](image-10.png)

