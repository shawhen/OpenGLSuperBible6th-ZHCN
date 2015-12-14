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

这个框架执行了一些初始化，最先调用`startup()`方法，然后在一个循环中调用`render()`方法。在缺省实现中，这些方法都定义为空的虚函数。我们在我们的派生类中重写`render()`方法，将我们的绘图代码写进去。应用框架负责创建一个窗体，处理输入，以及展示渲染结果给用户。我们的第一个示例的完整源代码如清单2.1，它的输出如图示2.1。

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
