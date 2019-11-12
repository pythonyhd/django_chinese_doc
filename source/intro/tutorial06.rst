===========================
开发第一个Django应用,Part6
===========================

本教程上接 :doc:`Tutorial 5 </intro/tutorial05>`。前面已经建立一个网页投票应用并且测试通过，现在主要讲述如何添加样式表和图片。

除由服务器生成的HTML文件外，网页应用一般还需要提供其它必要的文件——比如图片、JavaScript脚本和CSS样式表。
这样才能为用户呈现出一个完整的网站。 在Django中，这些文件统称为“ **静态文件** ”。

如果是在小型项目中，这只是个小问题，因为你可以将它们放在网页服务器可以访问到的地方。 但是呢，
在大一点的项目中 —— 尤其是由多个应用组成的项目，处理每个应用提供的多个静态文件集合还是比较麻烦的。

但是Django提供了 ``django.contrib.staticfiles``：它收集每个应用（和任何你指定的地方）的静态文件到一个单独的位置，
使得这些文件很容易维护。

自定义应用外观
==============

首先在 ``polls`` 路径中创建一个 ``static`` 目录。Django会从这里搜索静态文件，
这个和Django在 ``polls/templates/`` 中查找对应的模板文件的方式是一样的。

Django有一个 :setting:`STATICFILES_FINDERS` 的查找器，它会告诉Django从哪里查找静态文件。
其中有个内建的查找器 ``AppDirectoriesFinder`` ，它的作用是在
每个 :setting:`INSTALLED_APPS` 下查找“static”子目录下的静态文件。管理站点的静态文件也是使用相同的目录结构。

在你刚刚创建的static目录中，再创建一个 ``polls`` 目录并在它下面创建一个文件 ``style.css``。
这样你的 ``style.css`` 样式表应该在 ``polls/static/polls/style.css``。
因为根据 ``AppDirectoriesFinder`` 静态文件查找器的工作方式，
Django会在 ``polls/static`` 找到 ``polls/style.css`` 这个静态文件，和访问模板的路径类似。

.. admonition:: 静态文件命名空间

    和模板类似，其实我们也可以直接将静态文件直接放在 ``polls/static`` 下面（而不是再创建一个polls子目录），
    但是这样是一个不好的行为。Django会自动使用它所找到的第一个符合要求的静态文件的文件名，
    如果你有在两个不同应用中存在两个同名的静态文件，那么Django是无法区分它们的。
    所以我们需要告诉Django该使用其中的哪一个，最简单的方法就是为它们添加命名空间。
    也就是将这些静态文件放进以它们所在的应用的名字命名的子目录下。

样式表中写入这些内容( ``polls/static/polls/style.css`` ):

.. snippet:: css
    :filename: polls/static/polls/style.css

    li a {
        color: green;
    }

然后在 ``polls/templates/polls/index.html`` 中添加如下内容：

.. snippet:: html+django
    :filename: polls/templates/polls/index.html

    {% load static %}

    <link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}" />

``{%static%}`` 模板标签用于生成静态文件的绝对URL。

以上是你在开发过程中所需要对静态文件做的所有处理。 浏览器中重新载入 http://localhost:8000/polls/ ，
你应该会看到 ``Question`` 的超链接变成了绿色（Django的风格），这也表明你的样式表成功引入了。

添加背景图片
============

下一步，我们将创建一个子目录来存放图片。在 ``polls/static/polls/`` 目录中创建一个 ``images`` 子目录。
在这个目录中，放入一张图片 ``background.gif``。换句话，将你的图片放在 ``polls/static/polls/images/background.gif``。

然后，在样式表中添加（ ``polls/static/polls/style.css`` ）：


.. snippet:: css
    :filename: polls/static/polls/style.css

    body {
        background: white url("images/background.gif") no-repeat right bottom;
    }

重新加载 ``http://localhost:8000/polls/`` ， 你应该在屏幕的右下方看到载入的背景图片。

.. warning::

    ``{% static %}`` 模板标签在不是由 Django 生成的静态文件（比如样式表）中是不可用的。
    在以后开发过程中应该使用相对路径来相互链接静态文件，因为这样你可以只改变 :setting:`STATIC_URL`
    ( :ttag:`static` 模板标签用它来生成URLs)，而不用同时修改一大堆静态文件的路径。

这一上仅仅是 **基础** 。有关框架中包含的设置和其他更多详细信息，参见
:doc:`静态文件指南 </howto/static-files/index>` 和
:doc:`静态文件参考 </ref/contrib/staticfiles>`。:doc:`静态文件部署 </howto/static-files/deployment>` 讲述如何在真实的服务器上使用静态文件。

当您对静态文件掌握的差不多了时，请阅读本教程的 :doc:`第七部分 </intro/tutorial07>`，了解如何自定义Django自动生成的管理站点。