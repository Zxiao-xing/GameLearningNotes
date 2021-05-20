**OpenGL**：一种现代化的、硬件无关的应用程序编程接口，可以对图形硬件设备特性进行访问的软件库。自身并不包含任何执行窗口任务或处理用户输入的函数，也没有提供任何用于表达三维物体模型或者读取图像文件的操作。使用C/S的形式实现，编写的应用程序可以看做客户端，而计算机图形硬件厂商所提供的的OpenGL实现可以看做服务端。且其是基于光栅化的系统，也有其他例如光线追踪的方法生成图像

**OpenGL程序需要执行的主要操作**：

1. 从OpenGL的几何图元中设置数据，用于构建形状
2. 使用不同的着色器对输入的图元数据执行计算操作，判断他们的位置、颜色以及其他渲染属性
3. 将输入图元的数学描述转换为与屏幕位置对应的像素片元（光栅化）
4. 若有必要，针对光栅化工程产生的每个片元执行一些额外的操作

**OpenGL语法**：

- 函数：所以 "gl" 为前缀，然后以一个或多个大写字母开头的词组命名。由于OpenGL是一个C语言形式的库，因此不能使用函数的重载来处理不同类型的数据，它使用细微变化的函数名来管理实现同一类功能的函数集

  - 以 "glCreate" 为前缀的函数负责匹配不同类型的OpenGL对象的名称，其类似于一个指针变量，可以用来分配内存对象并且使用名称引用它。若是新创建的对象，则带有默认状态
  - 以 "glBind" 为前缀的函数负责绑定对象，绑定之后OpenGL将其作为当前对象，后续所有的操作都会作用与当前对象
  - 以 "glDelete" 为前缀的函数负责释放对象
  - 以 "glIs" 为前缀的函数负责判定

- 常量：以GL_为前缀，使用下划线来分割单词，这些常量都是以#define完成的，可以在OpenGL头文件的glcorearb.h和glext.h中找到

- 类型：为了在操作系统之间一直OpenGL程序，还定义了专门的数据类型，如果使用C语言的数据类型来直接表示OpenGL数据时，可能因为OpenGL自身的实现不同造成类型不匹配

- 命令后缀与参数数据类型

  | 后缀 | 数据类型       | 对应的C数据类型 | 对应的OpenGL类型          |
  | ---- | -------------- | --------------- | ------------------------- |
  | b    | 8位整型        | signed char     | CLbyte                    |
  | s    | 16位整型       | signed short    | GLshort                   |
  | i    | 32位整型       | int             | GLint、Glsizei            |
  | f    | 32位浮点型     | float           | GLfloat、GLclampf         |
  | d    | 64位浮点型     | double          | GLdouble、CLclampd        |
  | ub   | 8位无符号整型  | unsigned char   | GLubyte                   |
  | us   | 16位无符号整型 | unsigned short  | GLushort                  |
  | ui   | 32位无符号整型 | unsigned int    | GLint、GLenum、GLbitfield |

**GLFW**：抽象化窗口管理和其他系统任务的开发库，函数以glfw开头

**OpenGL渲染管线**：一系列数据处理过程，并且将应用程序的数据转换到最终渲染的图像。

1. 接受用户提供的几何数据（顶点和几何图元）
2. 输入到一系列着色器阶段中进行处理，包括顶点着色、细分着色（包含两个着色器）、几何着色
3. 送入光栅化单元，对所有剪切区域内的图元生成片元数据
4. 对每个生成的片元执行片元着色器

其中顶点着色器和片元着色器是必须的，细分和几何着色器是非必须的

1. 准备传输数据：OpenGL将所有数据都保存到缓存对象中，相当于由OpenGL维护的一块内存区域
2. 将数据传输至OpenGL：当缓存初始化完毕后，调用OpenGL的绘制命令来请求渲染几何图元，通常是将顶点数据传输到OpenGL服务端
3. 顶点着色：对于绘制命令传输的每个顶点，OpenGL都会调用一个顶点着色器来处理相关的数据。通常来说一个复杂的应用程序可能包含很多顶点着色器，但在同一个时刻只能有一个着色器起作用。只是将数据复制并传递到下一个着色阶段叫做传递着色器
4. 细分着色：使用面片来描述一个物体的形状，并且使用相对简单的面片几何体连接来完成细分的工作，其结果是几何图元的数量增加，并且模型的外观会变得更平滑，该阶段使用两个着色器分别管理面片和生成最终的形状
5. 几何着色：允许在光栅化之前对每个几何图元做更进一步的处理
6. 图元装配：将顶点与相关的几何图元之间组织起来
7. 剪切：更新图元，保证不绘制视口之外的图元。由OpenGL自动完成
8. 光栅化：判断某一部分几何体所覆盖的屏幕空间，得到屏幕空间信息以及输入的顶点数据之后，直接对片元着色器的每个可变变量进行线性插值，然后将结果值传递给用户的及片元着色器。片元是一个“候选的像素”，可以放置在帧缓存中，但可能最终被剔除。光栅化是一个片元生命的开始
9. 片元着色：使用片元着色计算片元的颜色和深度值
10. 逐片元操作：使用深度测试和模板测试等来决定一个片元是否可见。当片元通过了所有激活的测试，就可以被直接绘制到帧缓存中，对应的像素的颜色值（可能也包括深度值）会被更新，若开启了融混模式，片元的颜色会与对应的像素的颜色相叠加，形成一个新的颜色值并写入帧缓存中。在纹理阶段可以从一张至多张纹理贴图中查找所需的像素数据值

**规格化设备坐标系统**：具有限制OpenGL绘制几何图元范围的坐标系统

**OpenGL初始化过程**：

1. 初始化顶点数组对象

   顶点数组对象：负责保存一系列的顶点数据，这些数据保存到缓存对象中，并且由当前绑定的顶点数组对象管理。只有一种顶点数组对象类型，有很多类型的对象，且其中一部分对象并不负责处理顶点数据。

   - 调用glCreateVertexArrays()分配顶点数组对象名称
   - 调用glBindVertexArray()绑定顶点对象到当前环境
   - 完成顶点对象操作后，调用glDeleteVertexArrays()释放当前对象
   - 调用glIsVertexArray()检查某个名称是否已经被保留为顶点数组对象

2. 分配缓存对象

   缓存对象：OpenGL服务端分配和管理的一块内存区域，几乎所有传入OpenGL的数据都是存储在缓存对象中，所以OpenGL有很多种缓存对象，在绑定缓存对象时需要指定对应的类型（该类型也称绑定目标）

   - 调用glCreateBuffers()创建顶点缓存对象名称
   - 调用glBIndBuffer绑定到当前OpenGL环境
   - 完成操作后，调用glDeleteBuffers()释放缓存对象
   - 使用glIsBuffer()判断一个整数数值是否是一个缓存对象名称

3. 将数据载入缓存对象

   - 调用glNamedBufferStorage()分配顶点缓存对象的空间并把顶点数据从对象传输到缓存中

4. 初始化顶点与片元着色器

   - 编写着色器
   - 调用glVertexAttribPointer()将着色器关联到顶点属性数组
   - 调用glEnableVertexAttribArray()启用顶点属性数组
   - 调用glIsEnabled()判断是否启用对应模式

### 示例

**trangles.cpp**

```c++
#include<glad/glad.h>
#include<GLFW/glfw3.h>

#include<iostream>
#define WIN32
enum VAO_IDs
{
	Triangles, NumVAOs
};

enum Buffer_IDs
{
	ArrayBuffer, NumBuffers
};

enum Attrib_IDs {
	vPosition = 0
};

typedef struct {
	GLenum type;
	const char* filename;
	GLuint shader;
}ShaderInfo;

GLuint VAOs[NumVAOs];
GLuint Buffers[NumBuffers];

const GLuint NumVertices = 6;

static const GLchar* ReadShader(const char* filename) {
#ifdef WIN32
	FILE* infile;
	int err = fopen_s(&infile, filename, "rb");
	if (err != 0) {
		std::cerr << "Unable to open file " << filename << std::endl;
		return NULL;
	}
#else
	FILE* infile = fopen(filename, "rb");

	if (infile == NULL) {
		std::cerr << "Unable to open file " << filename << std::endl;
		return NULL;
	}
#endif


	fseek(infile, 0, SEEK_END);
	int len = ftell(infile);
	fseek(infile, 0, SEEK_SET);

	GLchar* source = new GLchar[len + 1];

	fread(source, 1, len, infile);
	fclose(infile);

	source[len] = 0;

	return const_cast<const GLchar*>(source);
}

GLuint LoadShaders(ShaderInfo* shaders) {
	if (shaders == NULL)
		return 0;

	GLuint program = glCreateProgram();

	ShaderInfo* entry = shaders;
	while (entry->type != GL_NONE) {
		GLuint shader = glCreateShader(entry->type);
		entry->shader = shader;

		const GLchar* source = ReadShader(entry->filename);
		if (source == NULL) {
			for (entry = shaders; entry->type != GL_NONE; ++entry) {
				glDeleteShader(entry->shader);
				entry->shader = 0;
			}
			return 0;
		}

		glShaderSource(shader, 1, &source, NULL);
		delete[] source;

		glCompileShader(shader);

		GLint compiled;
		glGetShaderiv(shader, GL_COMPILE_STATUS, &compiled);
		if (!compiled) {
#ifdef _DEBUG
			GLsizei len;
			glGetShaderiv(shader, GL_INFO_LOG_LENGTH, &len);

			GLchar* log = new GLchar[len + 1];
			glGetShaderInfoLog(shader, len, &len, log);
			std::cerr << "Shader compilation failed: " << log << std::endl;
			delete[] log;
#endif /* DEBUG */

			return 0;
		}

		glAttachShader(program, shader);

		++entry;
	}

	glLinkProgram(program);

	GLint linked;
	glGetProgramiv(program, GL_LINK_STATUS, &linked);
	if (!linked) {
#ifdef _DEBUG
		GLsizei len;
		glGetProgramiv(program, GL_INFO_LOG_LENGTH, &len);

		GLchar* log = new GLchar[len + 1];
		glGetProgramInfoLog(program, len, &len, log);
		std::cerr << "Shader linking failed: " << log << std::endl;
		delete[] log;
#endif /* DEBUG */

		for (entry = shaders; entry->type != GL_NONE; ++entry) {
			glDeleteShader(entry->shader);
			entry->shader = 0;
		}

		return 0;
	}

	return program;
}

void init() {
	static const GLfloat vertices[NumVertices][2] = {
		{-0.9, -0.9},		//Triangle 1
		{0.85, -0.9},
		{-0.9, 0.85},
		{0.9, -0.85},		//Triangle 2
		{0.9, 0.9},
		{-0.85, 0.9}
	};

	glCreateBuffers(NumBuffers, Buffers);
	glNamedBufferStorage(Buffers[ArrayBuffer], sizeof(vertices), vertices, 0);

	ShaderInfo shaders[] = {
		{GL_VERTEX_SHADER, "triangles.vert"},
		{GL_FRAGMENT_SHADER, "triangles.frag"},
		{GL_NONE, NULL}
	};

	GLuint program = LoadShaders(shaders);
	glUseProgram(program);

	glGenVertexArrays(NumVAOs, VAOs);
	glBindVertexArray(VAOs[Triangles]);
	glBindBuffer(GL_ARRAY_BUFFER, Buffers[ArrayBuffer]);
	glVertexAttribPointer(vPosition, 2, GL_FLOAT, GL_FALSE, 0, 0);

	glEnableVertexAttribArray(vPosition);
}

void display() {
	static const float black[] = { 0.0f, 0.0f, 0.0f, 0.0f };
	glClearBufferfv(GL_COLOR, 0, black);

	glBindVertexArray(VAOs[Triangles]);
	glDrawArrays(GL_TRIANGLES, 0, NumVertices);
}

int main() {
	glfwInit();
	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 5);
	glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);

	GLFWwindow* window = glfwCreateWindow(640, 480, "Triangles", NULL, NULL);


	glfwMakeContextCurrent(window);

	if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) {
		std::cerr << "Failed to initialize GLAD" << std::endl;
		return -1;
	}

	init();

	while (!glfwWindowShouldClose(window)) {
		display();
		glfwSwapBuffers(window);
		glfwPollEvents();
	}

	glfwDestroyWindow(window);

	glfwTerminate();

	return 0;

}
```

