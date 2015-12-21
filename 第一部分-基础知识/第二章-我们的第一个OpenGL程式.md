# 第二章 我们的第一个OpenGL程式
## 我们会从本章学到什么
- 如何创建并编译着色器代码
- 如何使用OpenGL绘图
- 如何使用本书的应用框架来初始化我们的程式并进行清理

在本章中，我们引入本书中几乎所有示例都会使用的一个简单的应用框架。本章会向我们展示如何使用书中的应用框架创建主窗口并渲染简单图形到上面。我们还会看到一个很简单的GLSL着色器是怎样的，如何编译它，以及如何用它来渲染简单的点。本章包含了我们第一个非常简单的OpenGL三角形。

## 创建一个简单的应用
我们从一个超级简单的示例应用开始介绍本书其他部分将会使用到的应用框架。当然，若要编写一个大型的OpenGL程式你可以不使用我们的框架-事实上，这种情况下我们也不推荐它，它太简陋。不过，它简化了一些事情，可以让我们更快地开始编写OpenGL代码。

通过包含`sb7.h`到源代码中我们引入应用框架到我们的应用中。这是一个c++头文件，它定义了一个名为`sb7`的命名空间，这个命名空间中包含一个名为`sb7::aplication`的类的声明，我们可以在我们的示例中继承它。这个框架中还包含了一些工具函数以及一个称为`vmath`的简单的数学库，以期帮助我们解决OpenGL中的琐碎事务。

要创建一个应用，我们包含`sb7.h`，从`sb7::application`派生出一个类，并在我们的一个源文件中，包含宏`DECLARE_MAIN`。这个宏定义了我们应用的主入口(译者注:这个宏展开就是main函数啦)，它创建我们从`sb7::application`派生类的实例(我们把这个类作为这个宏的参数传入)并调用这个实例的`run()`方法，`run`方法实现了应用的主循环。

这个框架执行了一些初始化，最先调用`startup()`方法，然后在一个循环中调用`render()`方法。在缺省实现中，这些方法都定义为空的虚函数。我们在我们的派生类中覆盖`render()`方法，将我们的绘图代码写进去。应用框架负责创建一个窗体，处理输入，以及展示渲染结果给用户。我们的第一个示例的完整源代码如清单2.1，它的输出如图示2.1。

清单2.1: 我们的第一个OpenGL应用
	
	// Include the "sb7.h" header file
	#include "sb7.h"
	
	// Derive my_application from sb7::application
	class my_application : public sb7::application
	{
	public:
		// Our rendering function
		void render(double currentTime)
		{
			// Simply clear the window with red
			static const GLfloat red[] = { 1.0f, 0.0f, 0.0f, 1.0f };
			glClearBufferfv(GL_COLOR, 0, red);
		}
	};
	
	// Our one and only instance of DECLARE_MAIN
	DECLARE_MAIN(my_application); 
	
图示2.1

![figure 1.2](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible6th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure2.1.png)

清单2.1所示的示例简单地将整个主窗口清除为红色。这引入我们的第一个OpenGL函数: `glClearBufferfv()`。这个函数的原型为:

	void glClearBufferfv(GLenum buffer, Glint drawBuffer, const GLfloat * value);
	
所有的OpenGL函数都始于*gl*前缀，跟随着几个命名约定，比如将一些参数类型编码后做为函数名称的后缀。这样可以实现一种有限的重载形式，即使在那些不直接支持重载的语言中。在本例中，后缀*fv*表示这个函数使用一组(vector*v*)浮点值(float-point*f*)，在OpenGL中数组(arrays，在类似C的语言中通过指针进行引用)和向量(vector)是等价的。

`glClearBufferfv()`函数告诉OpenGL要清除第一个参数指定的缓冲区(本例中即GL_COLOR)为第三个参数指定的值。第二个参数，*drawBuffer*，在有多个输出缓冲区可被清除时使用。因为我们在这只使用一个缓冲区并且*drawBuffer*是一个从0开始的索引值，本例中我们将其设置为0即可。我们在这使用数组*red*来存储颜色，它包含四个浮点值-依次代表红(red)，绿(green)，蓝(blue)以及不透明度(alpha)。

红、绿、蓝顾名思义。alpha是一个颜色的第四个组成部分，常用来表示一个片段(fragment)的不透明性(*opacity*)。将alpha设置为0会使得片段完全透明，设置为1则使得它完全不透明。alpha值同样可以被存储在输出图像中并在OpenGL的某些地方被使用，即使我们看不到它。可以看到我们将红色和alpha都设置为1，其他的为0。这表示一个不透明的红色。这个应用的运行结果如图示2.1。

这个刚起步的应用不是特别有趣，它做的所有事情就是把窗口填为实心的红色。我们注意到`render()`函数接收一个参数-`currentTime`。这个参数表示从应用开始以来所经过的秒钟数，我们可以用它来创建一个简单的动画。本例中，我们可以用它来改变清除窗体的颜色。经过我们修改过之后的`render()`函数如清单2.2。

	// Our rendering function
	void render(double currentTime)
	{
		const GLfloat color[] = {(float)sin(currentTime) * 0.5f + 0.5f,
									(float)cos(currentTime) * 0.5f + 0.5f,
									0.0f, 1.0f };
		glClearBufferfv(GL_COLOR, 0, color);
	}
	
现在我们的窗体从红色渐变为黄色、橙色、绿色，然后再次从红色开始。呃...虽然还不是很High，但至少可以干点啥了。

## 使用着色器
正如我们在第一章提到过的图形管线，OpenGL通过连接多个叫做着色器的小程序并佐以固定功能函数作为"胶水"来工作。当我们绘图时，图形处理器执行我们的着色器并将它们的输入输出在管线中串联起来，直到像素完成于管线末端。不管要绘制点什么，我们都需要至少编写一对着色器。

OpenGL着色器使用一种叫做OpenGL着色语言(OpenGL Shading Language)的语言进行编写，或者叫做GLSL。这个语言起源于C，不过长久以来都在修改以更好地适用于图形处理器。所以如果你对C比较熟，那学习GLSL也是很容易的。这个语言的编译器内置在OpenGL中。我们编写的着色器的源代码放入到*着色器对象(shader object)*中并进行编译，然后多个着色器对象链接在一起形成一个*程式对象(program object)*。每个程式对象都可以包含多个不同阶段的着色器(译者注: 原文为shader stages)。OpenGL中的着色器的阶段有*顶点着色器(vertex shaders)*、*镶嵌控制着色器(tessellation control shader，或者叫曲面细分控制着色器)*、*镶嵌运算着色器(tessellation evaluation shader)，或者叫曲面细分运算着色器*、*几何着色器(geometry shaders)*、*片段着色器(fragment shaders)*以及*运算着色器(compute shaders)*。最小可用的管线配置只需要一个顶点着色器(或者就一个运算着色器)，但如果我们想在显示屏上看到点什么，那还需要一个片段着色器。

清单2.3 我们的第一个顶点着色器:

    #version 450 core
    
    void main(void)
    {
        gl_Position = vec4(0.0, 0.0, 0.5, 1.0);
    }

清单2.3展示了我们的第一个顶点着色器，它简单到不行。第一行，我们用`#version 450 core`声明想要着色器编译器使用着色语言的4.5版本。值得注意的是*core*表示我们只想用OpenGL核心档案所支持的特性。

接着我们声明了`main`函数，和C语言一样，这是着色器执行的入口点，和正经的C有区别的是GLSL的`main`函数没有`int`返回值和`int argc, char* argv[]`的参数。在我们的`main`函数中，我们赋了一个值给`gl_Position`，它是贯穿OpenGL所有着色器的纽带的一部分。所有以`gl_`开头的变量都是OpenGL的一部分并且与其他着色器或者OpenGL的某些固定功能函数相连。在顶点着色器中，`gl_Position`表示顶点的输出位置。我们使用的值`vec4(0.0, 0.0, 0.5, 1.0)`将顶点放在OpenGL*裁剪空间(clip space)*的中间，裁剪空间是OpenGL管线下一个阶段的坐标系统。

清单2.4 我们的第一个片段着色器:

    #version 450 core
    out vec4 colour;
    void main(void)
    {
        color = vec4(0.0, 0.8, 1.0, 1.0);
    }

清单2.4再次向我们展示了一个简单到不行的着色器，这次是一个片段着色器。同样，它始于4.5核心档案声明。然后它用**out**关键字来声明`color`为一个输出变量。在片段着色器中，输出变量的值会被发送到窗体或者显示屏。在`main`函数中给这个输出变量赋了值。缺省情况下，那个值会直接显示到显示屏上，那个值是四个浮点值的向量，四个浮点值分别表示红、绿、蓝以及alpha，一如`glClearBufferfv()`的参数。在这个着色器中，我们使用的`vec4(0.0, 0.8, 1.0, 1.0f)`是蓝绿色。

现在我们有了一个顶点着色器和片段着色器，是时候编译它们并一起链接到一个程式中供OpenGL运行。这跟C++程式或者其他类似语言编译然后链接产生可执行文件一样。用来链接多个着色器到一个程式对象的代码如清单2.5。

清单2.5 编译简单的着色器:

    GLuint compile_shaders(void)
    {
        GLuint vertex_shader;
        GLuint fragment_shader;
        GLuint program;
        
        // Source code for vertex shader
        static const GLchar* vertex_shader_source[] = 
        {
            "#version 450 core                                    \n"
            "                                                     \n"
            "void main(void)                                      \n"
            "{                                                    \n"
            "    gl_Position = vec4(0.0, 0.0, 0.5, 1.0);          \n"
            "}                                                    \n"
        };
        
        // Source code for fragment shader
        static const GLchar* fragment_shader_source[] = 
        {
            "#version 450 core                                    \n"
            "                                                     \n"
            "out vec4 color;                                      \n"
            "                                                     \n"
            "void main(void)                                      \n"
            "{                                                    \n"
            "    color = vec4(0.0, 0.8, 1.0, 1.0);                \n"
            "}                                                    \n"
        };
        
        // Create and compile vertex shader
        vertex_shader = glCreateShader(GL_VERTEX_SHADER);
        glShaderSource(vertex_shader, 1, vertex_shader_source, NULL);
        glCompileShader(vertex_shader);
        
        // Create and compile fragment shader
        fragment_shader = glCreateShader(GL_FRAGMENT_SHADER);
        glShaderSource(fragment_shader, 1, fragment_shader_source, NULL);
        glCompileShader(fragment_shader);
        
        // Create program, attach shaders to it, and link it
        program = glCreateProgram();
        glAttachShader(program, vertex_shader);
        glAttachShader(program, fragment_shader);
        glLinkProgram(program);
        
        // Delete the shaders as the program has them now
        glDeleteShader(vertex_shader);
        glDeleteShader(fragment_shader);
        
        return program;
    }
    
在清单2.5中，我们引入了几个新函数:

- **glCreateShader()** 创建一个空的着色器对象，准备接受源代码以及编译。
- **glShaderSource()** 将着色器源代码传递给着色器对象，这样着色器对象可以保留源代码的一份拷贝。
- **glCompileShader()** 编译着色器对象包含的所有源代码。
- **glCreateProgram()** 创建一个程式对象，然后可以附加着色器对象给它。
- **glAttachShader()** 附加一个着色器对象到一个程式对象上。
- **glLinkProgram()** 将一个程式对象上的所有着色器对象链接到一起。
- **glDeleteShader()** 删除一个着色器对象。一旦一个着色器对象被链接到一个程式对象中之后，这个程式对象就包含了相应的二进制代码，于是相关的着色器对象就不再需要了。

清单2.3和清单2.4的着色器源代码在我们的程式中被当做常量字符串传递给**glShaderSource()**函数，这个函数将这些常量字符串拷贝到我们用**glCreateShader()**创建的着色器对象中。着色器对象保存了它的源代码的一份拷贝，然后，当我们调用**glCompileShader()**时，着色器对象的GLSL源代码被编译为一种中间的二进制表示形式，它也同样被存储在着色器对象中。程式对象表示链接后会被用来进行渲染工作的可执行体。我们使用**glAttachShader()**将我们的着色器对象附加到程式对象上，然后调用**glLinkProgram()**将所有着色器对象链接到可被图形处理器运行的代码中。将一个着色器对象附加到一个程式对象上会创建一个到着色器对象的引用，然后我们可以删除这个着色器对象，因为这个程式对象在它需要的情况下会把持这个着色器对象的内容。清单2.5中的`compile_shaders`函数新创建的程式对象。当我们调用这个函数后我们需要将返回的程式对象在某个地方保存下来，这样我们就可以用它来绘图。同时，我们也不会想每次想用到这个程式对象的时候都对它进行重新编译。所以，我们需要一个程式启动时会且只会被调用一次的函数。*sb7*应用框架提供这样一个函数: `application::startup()`，我们可以在我们的示例应用中覆盖它来完成所有一次性的设置工作。

在我们可以绘制任何东西之前尚有一事需要处理，创建一个*顶点集对象(vertex array object - VAO)*，这个对象表现了OpenGL管线的顶点获取阶段并且用来给顶点着色器提供输入数据。现在我们的顶点着色器没有任何输入，我们不需要对VAO做什么。然而，我们还是需要创建VAO，这样OpenGL才会允许我绘图。我们调用OpenGL函数**glCreateVertexArrays()**来创建VAO，使用**glBindVertexArray()**将其附加到我们的上下文(context)中(译者注: **glCreateVertexArray()**函数从OpenGL 4.5才可用，笔者使用**glGenVertexArrays()**来进行实做)。它们的原型如下:

    void glCreateVertexArrays(GLsizei n, GLuint* arrays);
    
    void glGenVertexArrays(GLsizei n, GLuint* arrays);
    
    void glBindVertexArray(GLuint array);
    
顶点集对象维护了OpenGL管线输入数据的所有相关状态。我们在`startup()`函数中加入对**glCreateVertexArrays()**和**glBindVertexArray()**的调用。随着我们对OpenGL进行更多的学习我们将会熟悉这一模式。在OpenGL中很多东西都是用对象(objects)来表达的(比如顶点集对象)。我们用一个创建函数(比如**glCreateVertexArrays()**)来创建对象，并且用一个绑定函数(比如**glBindVertexArray()**)来绑定它们使得OpenGL知道我们想在上下文中使用它们。

清单2.6 创建成员变量:

    class my_application : public sb7::application
    {
    public:
        // <snip>
        
        void startup()
        {
            rendering_program = compile_shaders();
            glCreateVertexArrays(1, &vertex_array_object);
            glBindVertexArray(vertex_array_object);
        }
        
        void shutdown()
        {
            glDeleteVertexArrays(1, &vertex_array_object);
            glDeleteProgram(rendering_program);
            glDeleteVertexArrays(1, &vertex_array_object);
        }
        
    private:
        GLuint rendering_program;
        GLuint vertex_array_object;
    };

在清单2.6中，我们覆盖了`sb7::application`类的`startup()`成员函数并将我们自己的初始化代码放进去。再次提醒一下，和`render()`成员函数一样，`startup()`成员函数在`sb7::application`中被定义为一个空的虚函数(virtual function)并且在`run()`函数中被自动调用。我们在`startup()`汉中调用`compile_shaders`并将返回的程式对象存储到我们的应用类的`rendering_program`成员变量中。当我们的应用运行完成后，我们应当将这些资源清理干净。所以我们还覆盖了`shutdown()`成员函数，在其中删除掉我们在启动时创建的程式对象。一如我们使用完着色器对象后调用**glDeleteShader()**，我们在使用完程式对象后调用**glDeleteProgram()**。在我们的`shutdown()`函数中，我们还删除掉在`startup()`中创建的顶点集对象。

现在我们有了一个程式，我们需要在里面执行着色器并开始在显示屏上实际绘制些什么。我们修改之前的`render()`函数，调用**glUseProgram()**来指示OpenGL使用我们的程式对象进行渲染，然后再调用我们的第一个绘图命令: **glDrawArrays()**。更新过的代码如清单2.7:

清单2.7 渲染一个点:

    // Our rendering function
    void render(double currentTime)
    {
        const GLfloat color[] = { (float)sin(currentTime) * 0.5f + 0.5f,
                                  (float)cos(currentTime) * 0.5f + 0.5f,
                                  0.0f, 1.0f };
        glClearBufferfv(GL_COLOR, 0, color);
        
        // Use the program object we created earlier for rendering
        glUseProgram(rendering_program);
        
        // Draw one point
        glDrawArrays(GL_POINTS, 0, 1);
    }

**glDrawArrays()**函数将顶点发送到OpenGL管线。它的原型为:

    void glDrawArrays(GLenum mode, Glint first, GLsizei count);
    
每一个顶点都会经过顶点着色器(就像清单2.3中的那个)的洗礼。**glDrawArrays()**的第一个参数是`mode`，这个参数指示OpenGL我们想要渲染何种图元。因为我们只想绘制一个点，所以本例中我们指定为`GL_POINTS`。第二个参数`first`跟本例无关，我们设置为0就好了。最后一个参数是我们要渲染的顶点的个数。一个点用一个顶点表示，所以我们指示OpenGL只渲染一个顶点，于是我们看到一个点被渲染出来。程式的运行结果如图示2.2。

图示2.2 渲染一个点:

![figure2.2](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible6th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure2.2.png)

眼睛睁大大，可以看到窗体的正中间有一个小点。恭喜，我们已经完成了我们的第一个OpenGL渲染。尽管它不是很令人印象深刻，但它为我们之后进行更有趣的渲染打好了基础并它证明了我的应用框架和我们的简单的着色器可以正常工作。

为了让上面的点更加容易看到一点，我们可以让OpenGL绘制一个比单个像素要大一些的点。我们可以用**glPointSize()**函数来达成这个目的，它的原型为:

    void glPointSize(GLfloat size);

这个函数将点的直径设置为`size`个像素。绘制点的最大直径值是由OpenGL实现定义的。我们以后会深入这个话题和OpenGL常用功能之外的话题，但现在让我们依赖于一个事实，OpenGL保证最大支持的点得尺寸至少为64像素。将如下代码

    glPointSize(40.0f);
    
添加到清单2.7的渲染函数中，我们即设置点得直径为40像素了。输出结果如图示2.3。

图示2.3 绘制一个大一些的点:

![figure2.3](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible6th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure2.3.png)

译者大注:

本书译版示例代码: [github](https://github.com/shawhen/sb7examples.git)。

正如译者在第二章前奏中描述的一样，译者的机器上搭载的OpenGL最高规格为4.1版本，所以译者的代码都以OpenGL 4.1实做，但传达的思想我们应当能融会贯通。

## 绘制我们的第一个三角形
仅仅只是绘制一个很大的点并不是很带劲--我们已经提到过，OpenGL支持很多不同类型的图元，其中最重要的是点、线和三角形。在我们的玩具示例中我们将`GL_POINTS`传给**glDrawArrays()**函数，从而绘制了一个点。我们真正想要绘制的是线和三角形。你可能已经猜到，我们可以将`GL_LINES`或者`GL_TRIANGLES`传给**glDrawArrays()**，但有一个问题存在: 清单2.3中的顶点着色器将每一个顶点都放在裁剪空间的中间。对于绘制顶点来说，OpenGL为我们给点设置区域是挺好的。但对于线或者三角形来说，两个或者多个顶点放在同样的地方会导致*图元退化(degenerate primitive)*，一个零长度的线或者零面积的三角形。如果我们试着用这个着色器绘制除了点之外的任何东西，我们不会得到任何输出，因为所有的图元都退化(degenerate)了。要修正这个问题，我们需要修改顶点着色器使得它为每个顶点设置不同的位置。

幸运的是GLSL的顶点着色器包含一个名为`gl_VertexID`的特殊输入，它是着色器运行时正在被处理的顶点的索引。`gl_VertexID`输入值从**glDrawArrays()**的`first`参数开始计数并且每次向上计一个顶点直到`count`参数个顶点。`gl_VertexID`输入是GLSL提供的众多内置变量的其中一个，内置变量表示由OpenGL生成或者我们要在着色器中生成传给给OpenGL的数据。我们前面提到过的`gl_Position`也是一个内置变量。我们可以使用索引`gl_VertexID`来给每个顶点赋予不同的位置，下示清单2.8。

清单2.8 在一个顶点着色器中生成多个顶点:

    #version 450 core
    
    void main(void)
    {
        // Declare a hard-coded array of positions
        const vec4 vertices[3] = vec4[3](vec4(0.25, -0.25, 0.5, 1.0), vec4(-0.25, -0.25, 0.5, 1.0), vec4(0.25, 0.25, 0.5, 1.0));
        
        // Index into our array using gl_VertexID
        gl_Position = vertices[gl_VertexID];
    }
    
使用清单2.8中的着色器，我们即可基于每个顶点`gl_VertexID`的值而赋予不同的位置。数组`vertices`中的点构成一个三角形，并且我们将渲染函数中传给**glDrawArrays()**的`GL_POINTS`更变为`GL_TRIANGLES`，如清单2.9所示。

清单2.9 渲染一个三角形:

    //Our rendering function
    void render(double currentTime)
    {
        const GLfloat color[] = { 0.0f, 0.2f, 0.0f, 1.0f };

        glClearBufferfv(GL_COLOR, 0, color);
        
        // Use the program object we created earlier for rendering
        glUseProgram(rendering_program);
        
        // Draw one triangle
        glDrawArrays(GL_TRIANGLES, 0, 3);
    }
    
终得如下图像:

图示2.4 我们的第一个刚起步的OpenGL三角形:

![figure2.4](https://raw.githubusercontent.com/shawhen/OpenGLSuperBible6th-ZHCN/master/%E7%AC%AC%E4%B8%80%E9%83%A8%E5%88%86-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/figures/figure2.4.png)

## 总结
本章我们大致了解了我们的第一个OpenGL程式的构成。我们很快就会了解到如何从应用中向着色器传递数据，如何传递我们的输入到顶点着色器中，如何在着色器阶段间传递数据，等等。在本章，我们简要介绍了*sb7*应用框架，编译一个着色器，清除窗体以及绘制点和三角形。我们还看到如何使用**glPointSize()**函数来设置点的大小以及第一个绘图命令--**glDrawArrays()**。
