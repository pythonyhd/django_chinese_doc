==========
模型字段
==========

.. module:: django.db.models.fields
   :synopsis: Built-in field types.

.. currentmodule:: django.db.models

本文档包含了Django提供的全部模型 :class:`Field` 包括
`字段选项`_ 和 `字段类型`_ 的API参考。

.. seealso::

    如果内建的字段不能满足你的需求, 你可以蚕食 `django-localflavor
    <https://github.com/django/django-localflavor>`_ (`documentation
    <https://django-localflavor.readthedocs.io/>`_), 其中包含对特定国家和文化的各种配套代码。

    同时, 也可以 :doc:`自定义模型字段
    </howto/custom-model-fields>`.

.. note::

    严格意义上来讲， Model定义在 :mod:`django.db.models.fields` 里面, 但为了使用方便，它们被导入到
    :mod:`django.db.models` 中；一般使用 ``from django.db import models`` 导入，
    然后使用 ``models.<Foo>Field`` 的形式使用字段。

.. _common-model-field-options:

字段选项
=========

下列参数是全部字段类型都可用的，而且都是可选择的。

``null``
--------

.. attribute:: Field.null

如果为 ``True``, Django 将把空值在数据库中存为 ``NULL`` ，默认为
is ``False`` 。

不要为字符串类型字段设置 :attr:`~Field.null` ，例如
:class:`CharField` 和 :class:`TextField` 。因为这样会把空字符串存为 ``NULL`` 。
如果字符串字段设置了 ``null=True`` ，那意味着对于“无数据”有两个可能的值： :attr:`~Field.null` 和空字符串。
这种明显是多余的，Django 的惯例是使用空字符串而不是NULL。

无论是字符串字段还是非字符串字段，如果你希望在表单中允许存在空值，必须要设置
``blank=True`` ，因为仅仅影响数据库存储。(参见 :attr:`~Field.blank`)。

.. note::

    在使用Oracle 数据库时，数据库将存储 ``NULL`` 来表示空字符串，而与这个属性无关。

如果你希望 :class:`BooleanField` 接受 :attr:`~Field.null` 值,
那么使用 :class:`NullBooleanField` 代替。

``blank``
---------

.. attribute:: Field.blank

如果为 ``True``, 则该字段允许为空。默认为 ``False`` 。

注意它和 :attr:`~Field.null` 不同。 :attr:`~Field.null` 纯粹是数据库范畴的概念，而
:attr:`~Field.blank` 是数据验证范畴。 如果字段设置 ``blank=True``,
表单验证时将允许输入空值。如果字段设置 ``blank=False``, 则该字段为必填。

.. _field-choices:

``choices``
-----------

.. attribute:: Field.choices

它是一个可迭代的结构(e.g., 列表或元组) 。由可迭代的二元组组成
(e.g. ``[(A, B), (A, B) ...]``)，
用来给字段提供选择项。如果设置了 ``choices`` ，
默认表格样式就会显示选择框，而不是标准的文本框，而且这个选择框的选项就是 ``choices`` 中的元组。

每个元组中的第一个元素，是存储在数据库中的值；第二个元素是该选项描述。 比如::

    YEAR_IN_SCHOOL_CHOICES = (
        ('FR', 'Freshman'),
        ('SO', 'Sophomore'),
        ('JR', 'Junior'),
        ('SR', 'Senior'),
    )

一般来说，最好在模型类内部定义choices，然后再给每个值定义一个合适名字::

    from django.db import models

    class Student(models.Model):
        FRESHMAN = 'FR'
        SOPHOMORE = 'SO'
        JUNIOR = 'JR'
        SENIOR = 'SR'
        YEAR_IN_SCHOOL_CHOICES = (
            (FRESHMAN, 'Freshman'),
            (SOPHOMORE, 'Sophomore'),
            (JUNIOR, 'Junior'),
            (SENIOR, 'Senior'),
        )
        year_in_school = models.CharField(
            max_length=2,
            choices=YEAR_IN_SCHOOL_CHOICES,
            default=FRESHMAN,
        )

        def is_upperclass(self):
            return self.year_in_school in (self.JUNIOR, self.SENIOR)

当然也可以在模型类的外部定义choices然后引用它，
但是在模型类中定义choices和其每个choice的name(即元组的第二个元素)可以保存所有使用choices的信息，
也使得choices更容易被应用（例如， ``Student.SOPHOMORE`` 可以在任何引入 ``Student`` 模型的地方生效)。

也可以将选项归类到已命名的组中用来达成组织整理的目的::

    MEDIA_CHOICES = (
        ('Audio', (
                ('vinyl', 'Vinyl'),
                ('cd', 'CD'),
            )
        ),
        ('Video', (
                ('vhs', 'VHS Tape'),
                ('dvd', 'DVD'),
            )
        ),
        ('unknown', 'Unknown'),
    )

每个元组的第一个元素是组的名字。 第二个元素是一组可迭代的二元元组，
每一个二元元组包含一个值和一个适合人看的名字构成一个选项。
分组的选项可能会和未分组的选项合在同一个list中。
（就像例中的 `unknown` 选项）。

对于包含 :attr:`~Field.choices` 的模型字段, Django 将会加入一个方法来获取当前字段值的易于理解的名称
(即元组的第二个值)。参见数据库API文档中的
:meth:`~django.db.models.Model.get_FOO_display` 。

choices 只要是可迭代的对象就行 -- 并不一定要是列表和元组。这样你可以动态构建choices。
但是如果你自己搞不定动态的
:attr:`~Field.choices` 。你最好还是使用 :class:`ForeignKey` 来构建一个合适的数据库表。
而对于静态数据来说， :attr:`~Field.choices` 是不改变的。

除非在字段中设置了 :attr:`blank=False<Field.blank>` 否则将选择框中将包含一个 ``"---------"`` 的标签。
只要在添加一个  ``None`` 到 ``choices`` 元组中就可以改变这一行为。e.g. ``(None, 'Your String For Display')`` ，
或者, 你可以用一个空字符串代替 ``None``  比如在一个 :class:`~django.db.models.CharField` 。

``db_column``
-------------

.. attribute:: Field.db_column

数据库中储存该字段的列名。如果未指定，那么Django将会使用Field名作为字段名。

如果你的数据库列名是SQL语句的保留字，或者是包含不能作为Python 变量名的字符，如连字符。这也不会有问题，
Django 会在后台给列名和表名加上双引号。

``db_index``
------------

.. attribute:: Field.db_index

如果为 ``True``,将会为这个字段创建数据库索引。

``db_tablespace``
-----------------

.. attribute:: Field.db_tablespace

The name of the :doc:`database tablespace </topics/db/tablespaces>` to use for
this field's index, if this field is indexed. The default is the project's
:setting:`DEFAULT_INDEX_TABLESPACE` setting, if set, or the
:attr:`~Options.db_tablespace` of the model, if any. If the backend doesn't
support tablespaces for indexes, this option is ignored.

``default``
-----------

.. attribute:: Field.default

该字段的默认值. 它可以是一个值或者一个可调用对象。
如果是一个可调用对象，将会在创建新对象时调用。

这个默认值不可以是一个可变对象(比如 ``模型实例`` , ``列表`` , ``集合`` 等等)。
因为对于所有模型的一个新的实例来说，它们指向同一个引用。
或者，把他们封装为一个可调用的对象。
例如，有一个自定义的 :class:`~django.contrib.postgres.fields.JSONField`,想指定一个特定的字典值，可以这样::

    def contact_default():
        return {"email": "to1@example.com"}

    contact_info = JSONField("ContactInfo", default=contact_default)

``lambda`` 表达式不能作为 ``default`` 的参数值，因为她们无法 :ref:`被migrations序列化 <migration-serializing>` 。

对于将映射到模型实例的外键之类的字段，默认值应该是它们引用的字段的值(除非to字段设置为pk)，而不是模型实例。

对于将映射到模型实例的字段，例如 :class:`ForeignKey` 。其默认值应该是他们引用的字段值( 除非
:attr:`~ForeignKey.to_field` 设置了 ``pk``)而不是模型实例。

默认值会在新实例创建并且没有给该字段提供值时使用。如果字段为主键，默认值也会在设置为 ``None````None`` 时使用。

``editable``
------------

.. attribute:: Field.editable

如果设为 ``False`` , 这个字段将不会出现在admin或者其他 :class:`~django.forms.ModelForm` 中。
同时也会跳过 :ref:`模型验证 <validating-objects>` 。 默认是 ``True``。

``error_messages``
------------------

.. attribute:: Field.error_messages

 ``error_messages`` 参数用于重写默认的错误提示，通过字典的key来匹配将要改变的错误类型。

错误提示的key包括 ``null``, ``blank``, ``invalid``, ``invalid_choice``,
``unique``, 和 ``unique_for_date`` 。
下面的 `字段类型`_ 中的 error_messages 的 keys 是不一样的。

``help_text``
-------------

.. attribute:: Field.help_text

表单部件显示的额外"帮助"信息。即使字段不是在表单上使用，它也很有用。

注意它 *不会* 自动转义HTML标签，这样就可以在
 :attr:`~Field.help_text` 中定义html格式::

    help_text="Please use the following format: <em>YYYY-MM-DD</em>."

另外, 你可以使用简单文本和
``django.utils.html.escape()`` 来避免任何HTML特定的字符。
请确保你所使用的help text能够避免一些不信任用户的跨站脚本攻击。

``primary_key``
---------------

.. attribute:: Field.primary_key

如果是 ``True``, 则该字段会成为模型的主键字段。

如果模型中没有字段设置了 ``primary_key=True`` , Django
会自动添加一个 :class:`AutoField` 字段来作为主键，因此如果没有特定需要，可以不用设置
``primary_key=True`` 。
更多信息，请参考
:ref:`automatic-primary-key-fields` 。

``primary_key=True`` 也就意味着 :attr:`null=False <Field.null>` 和
:attr:`unique=True <Field.unique>` 。并且一个对象只能有一个主键。

主键字段是只读的，如果你修改了一条记录主键的值并且保存，那么将是创建一条新的记录而不是修改。

``unique``
----------

.. attribute:: Field.unique

如果为 ``True``, 改字段在表中必须唯一。

这是通过模型验证的数据库级别的强制性操作。如果你对 :attr:`~Field.unique`
字段保存重复的值，:meth:`~django.db.models.Model.save` 方法将回抛出
:exc:`django.db.IntegrityError` 异常。

这个选项适用于除了 :class:`ManyToManyField`,
:class:`OneToOneField`, :class:`FileField` 的其他所有字段。

当设置了 ``unique`` 为 ``True`` 后，可以不必再指定
:attr:`~Field.db_index`, 因为 ``unique`` 也会创建索引。

``unique_for_date``
-------------------

.. attribute:: Field.unique_for_date

设置为 :class:`DateField` 或者 :class:`DateTimeField` 字段的名字，表示要求该字段对于相应的日期字段值是唯一的。

例如，字段 ``title`` 设置了
``unique_for_date="pub_date"`` ，那么Django将不会允许在同一 ``pub_date`` 的两条记录的
``title`` 相同。

需要注意的是，如果你设置的是一个 :class:`DateTimeField`, 也只会考虑日期部分。
此外，如果设置了 :setting:`USE_TZ` 为 ``True``,
检查将以对象保存时的 :ref:`当前时区
<default-current-time-zone>` 进行。

这是在模型验证期间通过 :meth:`Model.validate_unique()` 强制执行的，
而不是在数据库层级进行验证。
如果 :attr:`~Field.unique_for_date` 约束涉及的字段不是
:class:`~django.forms.ModelForm` 中的字段(例如， ``exclude`` 中列出的字段或者设置了
:attr:`editable=False<Field.editable>`), :meth:`Model.validate_unique()` 将忽略该特殊的约束。

``unique_for_month``
--------------------

.. attribute:: Field.unique_for_month

和 :attr:`~Field.unique_for_date` 类似, 只是要求字段对于月份是唯一的。

``unique_for_year``
-------------------

.. attribute:: Field.unique_for_year

和 :attr:`~Field.unique_for_date` 、 :attr:`~Field.unique_for_month` 类似。

``verbose_name``
-------------------

.. attribute:: Field.verbose_name

为字段设置的可读性更高的名称。如果用户没有定义改选项，
Django会自动将自动创建，内容是该字段属性名中的下划线转换为空格的结果。可以参照
:ref:`Verbose field names <verbose-field-names>`.

``validators``
-------------------

.. attribute:: Field.validators

该字段将要运行的一个Validator的列表。 更多信息参见 :doc:`validators
文档</ref/validators>` 。

注册和查询
~~~~~~~~~~~~

``Field`` 实现了 :ref:`查询注册API <lookup-registration-api>` 。
该API 可以用于自定义一个字段类型的查询，以及如何从一个字段获取查询。

.. _model-field-types:

字段类型
==========

.. currentmodule:: django.db.models

``AutoField``
-------------

.. class:: AutoField(**options)

一个根据实际ID自动增长的 :class:`IntegerField` 。
通常不需要直接使用它,如果表中没有设置主键时，将会自动添加一个自增主键。参考
:ref:`automatic-primary-key-fields` 。

``BigAutoField``
----------------

.. class:: BigAutoField(**options)

.. versionadded:: 1.10

一个64位整数, 和 :class:`AutoField` 类似，不过它的范围是
``1`` 到 ``9223372036854775807`` 。

``BigIntegerField``
-------------------

.. class:: BigIntegerField(**options)

一个64位整数, 和 :class:`IntegerField` 类似，不过它允许的值范围是
``-9223372036854775808`` 到 ``9223372036854775807`` 。这个字段默认的表单组件是一个
:class:`~django.forms.TextInput`.

``BinaryField``
-------------------

.. class:: BinaryField(**options)

这是一个用来存储原始二进制码的字段。支持 ``bytes`` 赋值。 注意这个字段只有很有限的功能。
比如，不能在一个 ``BinaryField`` 值的上进行查询过滤。
:class:`~django.forms.ModelForm` 中也不允许包含 ``BinaryField`` 。

.. admonition:: 滥用 ``BinaryField``

    可能你想使用数据库来存储文件，但是99%的情况下这都是不好的设计。 这个字段 *不是* 代替
    :doc:`静态文件 </howto/static-files/index>` 的最佳方式。

``BooleanField``
----------------

.. class:: BooleanField(**options)

一个 true/false 字段。

这个字段的默认表单部件是
:class:`~django.forms.CheckboxInput` 。

如果要允许 :attr:`~Field.null` 值，那么可以使用
:class:`NullBooleanField` 代替。

如果 :attr:`Field.default` 没有指定，
 ``BooleanField`` 的默认值将是 ``None`` 。


``CharField``
-------------

.. class:: CharField(max_length=None, **options)

字符字段。

对于比较大的文本内容,请使用 :class:`~django.db.models.TextField` 类型。

这个字段的默认表单部件是 :class:`~django.forms.TextInput`.

:class:`CharField` 字段有个额外参数:

.. attribute:: CharField.max_length

    字段允许的最大字符串长度。这将在数据库中和表单验证时生效。

.. note::

    如果你在写一个需要导出到多个不同数据库后端的应用，你需要注意某些后端对 ``max_length`` 有一些限制，
    查看 :doc:`数据库后端注意事项 </ref/databases>` 来获取更多的细节。

.. admonition:: MySQL users

    如果你使用的是 MySQLdb 1.2.2 设置的是 ``utf8_bin`` 校对
    (这 *不是* 默认项), 你必须了解一些问题。详细请参考 :ref:`MySQL 数据库注意事项 <mysql-collation>` 。

``CommaSeparatedIntegerField``
------------------------------

.. class:: CommaSeparatedIntegerField(max_length=None, **options)

.. deprecated:: 1.9

    这个字段可以使用 :class:`~django.db.models.CharField`
    的 ``validators=[``\ :func:`validate_comma_separated_integer_list
    <django.core.validators.validate_comma_separated_integer_list>`\ ``]`` 代替。

一个逗号分隔的整数字段。 和 :class:`CharField` 一样，
:attr:`~CharField.max_length` 是必填参数，同时再数据库移植时也需要注意。

``DateField``
-------------

.. class:: DateField(auto_now=False, auto_now_add=False, **options)

日期, 表现为 Python中  ``datetime.date`` 的实例。但是有几个额外的设置参数:

.. attribute:: DateField.auto_now

    当保存对象时，自动设置改字段为当前时间。比如用于字段 "修改时间"。
    注意这会 *始终* 使用当前时间，并不是一个默认值。

    这个字段只会在调用 :meth:`Model.save()
    <django.db.models.Model.save>` 方法时自动更新。
    更新其他字段或使用这他更新方法时，这个字段不会更新，比如 :meth:`QuerySet.update()
    <django.db.models.query.QuerySet.update>`  方法。

.. attribute:: DateField.auto_now_add

    当字段被首次创建时被设置为当前时间。
    适用于创建日期，注意这会 *始终* 使用当前时间，并不是一个可以重写的默认值。
    因此就算你在创建时指定了日期，也会被忽略掉。如果你希望修改这个值，请设置以下内容，而不是
    ``auto_now_add=True``:

    * 对于 :class:`DateField`: ``default=date.today`` - from
      :meth:`datetime.date.today`
    * 对于 :class:`DateTimeField`: ``default=timezone.now`` - from
      :func:`django.utils.timezone.now`

这个字段的默认表单部件是
:class:`~django.forms.TextInput` 。在admin中会添加一个JavaScript的日期插件，带有一个
"Today" 的快捷按钮。 也包含 ``invalid_date`` 错误消息。

选项 ``auto_now_add``, ``auto_now``, 和 ``default`` 是相互排斥的，
他们之间的任何组合将会发生错误的结果。

.. note::
    在当前框架实现中, 设置 ``auto_now`` 或者 ``auto_now_add`` 为
    ``True`` 都会使字段同时得到两个设置 ``editable=False`` 和 ``blank=True`` 。

.. note::
    ``auto_now`` 和 ``auto_now_add`` 这两个设置会在对象创建或更新时，总是使用
    :ref:`default timezone <default-current-time-zone>` 的日期。
    你可以考虑一下简单地使用你自己的默认调用或者重写
    ``save()`` 方法来代替使用
    ``auto_now`` 和 ``auto_now_add``;也可以使用
    ``DateTimeField`` 类型代替 ``DateField`` ，然后在显示时间的时候再决定如何从
    datetime 转换到 date。

``DateTimeField``
-----------------

.. class:: DateTimeField(auto_now=False, auto_now_add=False, **options)

时间和日期, 表现为 Python 中的 ``datetime.datetime`` 实例。
和 :class:`DateField` 有相同的额外参数。

默认的表单部件是一个
:class:`~django.forms.TextInput` 。admin页面使用两个带有 JavaScript控件的
:class:`~django.forms.TextInput` 。

``DecimalField``
----------------

.. class:: DecimalField(max_digits=None, decimal_places=None, **options)

十进制数，表现为Python中
:class:`~decimal.Decimal` 实例，有两个 **必填** 参数:

.. attribute:: DecimalField.max_digits

    允许的最大位数包括小数点后的位数。必须大于等于
    ``decimal_places``。

.. attribute:: DecimalField.decimal_places

    小数点后的位数。

举个例子，要储存最大为 ``999`` 带有两位小数的数字，需要这样设置::

    models.DecimalField(..., max_digits=5, decimal_places=2)

而要存储那些将近10亿，并且要求达到小数点后十位精度的数字::

    models.DecimalField(..., max_digits=19, decimal_places=10)

该字段默认的表单部件是 :class:`~django.forms.NumberInput` ，
如果 :attr:`~django.forms.Field.localize` 设置为 ``False`` 则是
:class:`~django.forms.TextInput` 。

.. note::

    关于更多
    :class:`FloatField` 和 :class:`DecimalField` 的区别信息，请参考
    :ref:`FloatField vs. DecimalField <floatfield_vs_decimalfield>` 。

``DurationField``
-----------------

.. class:: DurationField(**options)

用作存储一段时间的字段类型 - 类似Python中的
:class:`~python:datetime.timedelta` 。.当数据库使用的是 PostgreSQL, 该数据类型使用的是一个 ``interval`` ，
而在Oracle上，则使用的是 ``INTERVAL DAY(9) TO
SECOND(6)`` 。其他情况使用微秒的的 ``bigint`` 。

.. note::

    ``DurationField`` 的算数运算在多数情况下可以很好的工作。
    然而，在除了 PostgreSQL之外的其他数据库中, 将 ``DurationField``
    和 ``DateTimeField`` 的实例比较则不会得到正确的结果。

``EmailField``
--------------

.. class:: EmailField(max_length=254, **options)

一个检查输入的email地址是否合法的 :class:`CharField` 类型。
它使用 :class:`~django.core.validators.EmailValidator` 来验证输入合法性。

``FileField``
-------------

.. class:: FileField(upload_to=None, max_length=100, **options)

上传文件字段。

.. note::
    ``primary_key`` 和 ``unique`` 在这个字段中不支持, 如果使用在抛出
    ``TypeError`` 异常。

拥有两个可选参数:

.. attribute:: FileField.upload_to

    用于设置文件路径和文件名,
    有两种设置方法，但都是通过
    :meth:`Storage.save() <django.core.files.storage.Storage.save>` 方法设置。

    你如果路劲包含 :func:`~time.strftime` 格式，将被替换为实际的 日期/时间 作为文件路径
    (这样上传的文件就不会到指定的文件夹)。 例如::

        class MyModel(models.Model):
            # file will be uploaded to MEDIA_ROOT/uploads
            upload = models.FileField(upload_to='uploads/')
            # or...
            # file will be saved to MEDIA_ROOT/uploads/2015/01/30
            upload = models.FileField(upload_to='uploads/%Y/%m/%d/')

    如果使用了默认的
    :class:`~django.core.files.storage.FileSystemStorage`, 这个字符串将会再添加
    上 :setting:`MEDIA_ROOT` 路径。如果想使用其他储存方法，请查看储存文档如何处理
    ``upload_to`` 。

    ``upload_to`` 也可以是一个可调用的对象。 将调用它来获取上传路径，包括文件名。
    这个可调用对象必须接受两个参数，并且返回一个Unix 风格的路径（带有前向/）给存储系统。将传递的两个参数为:

    ======================  ===============================================
    Argument                Description
    ======================  ===============================================
    ``instance``            定义 ``FileField`` 的模型的实例
                            更准确地说，就是当前文件的所在的那个实例。

                            大部分情况下，这个实例将还没有保存到数据库中，若它用到默认的
                            ``AutoField`` 字段，它的主键字段还可能没有值。

    ``filename``            最初给文件的文件名。在确定最终目标路径时，可能不会考虑到这一点。
    ======================  ===============================================

    例如::

        def user_directory_path(instance, filename):
            # file will be uploaded to MEDIA_ROOT/user_<id>/<filename>
            return 'user_{0}/{1}'.format(instance.user.id, filename)

        class MyModel(models.Model):
            upload = models.FileField(upload_to=user_directory_path)

.. attribute:: FileField.storage

    一个Storage 对象，用于你的文件的存取。参见 :doc:`/topics/files` 查看如何提供这个对象的细节。

这个字段的默认表单部件是一个
:class:`~django.forms.ClearableFileInput` 。

在模式中使用 :class:`FileField` 类型或 :class:`ImageField` (见下文) 需要如下几步:

1. 在 settings 文件中, 将Django储存文件的的完整路径设置到 :setting:`MEDIA_ROOT` 项。
   (从性能上考虑，不要将这些文件存在数据库中) 定义一个
   :setting:`MEDIA_URL` 作为基础目录的URL。确保web server账户对这个目录有读写权限。

2. 添加 :class:`FileField` 或者 :class:`ImageField` 到模型中, 定义
   属性 :attr:`~FileField.upload_to` 用来存放上传的文件，应该是
   :setting:`MEDIA_ROOT` 的子目录。

3. 数据库中存放的仅是这个文件的路径
   (相对于 :setting:`MEDIA_ROOT`)。 然后使用 :attr:`~django.db.models.fields.files.FieldFile.url` 属性
   来获取。例如, 一个叫做 ``mug_shot`` 的 :class:`ImageField` 字段，可以使用
   ``{{ object.mug_shot.url }}`` 来获取它的绝对路径。

假如, 你将 :setting:`MEDIA_ROOT` 设置为 ``'/home/media'``,
:attr:`~FileField.upload_to` 属性设置为 ``'photos/%Y/%m/%d'`` 。
:attr:`~FileField.upload_to` 的 ``'%Y/%m/%d'`` 部分是
:func:`~time.strftime` 的格式化字符。
``'%Y'`` 表示四位数的年份, ``'%m'`` 表示两位数的月份， ``'%d'`` 表示两位数的日份。
如果文件上传时间是 Jan. 15, 2007,将被存到目录 ``/home/media/photos/2007/01/15`` 。

如果想要知道上传文件的磁盘文件名或文件大小，
可以使用 :attr:`~django.core.files.File.name` 和
:attr:`~django.core.files.File.size` 属性得到；关于更多支持的方法和属性。请参考
:class:`~django.core.files.File` 的参考文档 :doc:`/topics/files` 。

.. note::
    文件作为模型存储在数据库中的一部分，磁盘上文件名在模型保存完毕之前是不可靠的。


上传的文件对应的URL可以通过使用
:attr:`~django.db.models.fields.files.FieldFile.url` 属性获得. 其实在内部，是调用
:class:`~django.core.files.storage.Storage` 类的 :meth:`~django.core.files.storage.Storage.url` 方法。

.. _file-upload-security:

值得注意的是，无论任何时候处理上传文件时，都应该密切关注文件将被上传到哪里，上传的文件类型，以避免安全漏洞。
*检查所有上传文件* 确保上传的文件是被允许的文件。
例如，如果你盲目的允许其他人在无需认证的情况下上传文件至你的web服务器的root目录中，
那么别人可以上传一个CGI或者PHP脚本然后通过访问一个你网站的URL来执行这个脚本。
所以，不要允许这种事情发生。

甚至是上传HTML文件也需要注意，它可以通过浏览器（虽然不是服务器）执行，也可以引发相当于是XSS或者CSRF攻击的安全威胁。

:class:`FileField` 对象在数据库中以最大长度100个字节的 ``varchar`` 类型存在。
和普通字段一样。也可以使用 :attr:`~CharField.max_length` 参数限制。

``FileField`` 和 ``FieldFile``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. currentmodule:: django.db.models.fields.files

.. class:: FieldFile

当添加一个 :class:`~django.db.models.FileField` 字段到模型中时，实际上是一个
:class:`FieldFile` 实例作为将要访问文件的代理。

The API of :class:`FieldFile` mirrors that of :class:`~django.core.files.File`,
with one key difference: *The object wrapped by the class is not necessarily a
wrapper around Python's built-in file object.* Instead, it is a wrapper around
the result of the :attr:`Storage.open()<django.core.files.storage.Storage.open>`
method, which may be a :class:`~django.core.files.File` object, or it may be a
custom storage's implementation of the :class:`~django.core.files.File` API.

In addition to the API inherited from
:class:`~django.core.files.File` such as :meth:`~django.core.files.File.read`
and :meth:`~django.core.files.File.write`, :class:`FieldFile` includes several
methods that can be used to interact with the underlying file:

.. warning::

    Two methods of this class, :meth:`~FieldFile.save` and
    :meth:`~FieldFile.delete`, default to saving the model object of the
    associated ``FieldFile`` in the database.

.. attribute:: FieldFile.name

文件的名称，:class:`~django.core.files.storage.Storage` 的根路径，
:class:`~django.db.models.FileField` 的相对路径。

.. attribute:: FieldFile.size

下面 :attr:`Storage.size()
<django.core.files.storage.Storage.size>` 方法的返回结果。

.. attribute:: FieldFile.url

通过下面 :class:`~django.core.files.storage.Storage` 类的
:meth:`~django.core.files.storage.Storage.url` 方法只读地访问文件的URL。

.. method:: FieldFile.open(mode='rb')

在指定 `模式` 下打开或重写与该实例相关联的文件，和Python的标准 ``open()``
方法不同的是，它不返回文件描述符。

由于底层文件在访问它时隐式地打开，因此调用此方法可能没有必要，除非将指针重置到底层文件或更改 `模式`。

.. method:: FieldFile.close()

类似Python的 ``file.close()`` 方法，关闭相关文件。

.. method:: FieldFile.save(name, content, save=True)

这个方法会将文件名以及文件内容传递到字段的storage类中,并将模型字段与保存好的文件关联。
I
如果想要手动关联文件数据到你的模型中的
:class:`~django.db.models.FileField` 实例,
则 ``save()`` 方法总是用来保存该数据。

方法接受两个必选参数: ``name`` 文件名, 和
``content`` 文件内容。  可选参数
``save`` 控制模型实例在关联的文件被修改时是否保存，默认为
``True`` 。

需要注意的是 ``content`` 参数必须是
:class:`django.core.files.File` 的实例, 不是Python内建的文件对象。
你可以用如下方法从一个已经存在的Python文件对象来构建 :class:`~django.core.files.File`::

    from django.core.files import File
    # Open an existing file using Python's built-in open()
    f = open('/path/to/hello.world')
    myfile = File(f)

或者，你可以像下面的一样从一个python字符串中构建::

    from django.core.files.base import ContentFile
    myfile = ContentFile("hello world")

其他更多信息, 参见 :doc:`/topics/files`.

.. method:: FieldFile.delete(save=True)

删除与此实例关联的文件，并清除该字段的所有属性。
注意:如果在调用 ``delete()`` 时有打开操作，该方法将关闭该文件。

``save`` 控制模型实例在关联的文件被修改时是否保存，默认为
``True`` 。

注意，model删除的时候，与之关联的文件并不会被删除。如果你要把文件也清理掉，你需要自己处理

.. currentmodule:: django.db.models

``FilePathField``
-----------------

.. class:: FilePathField(path=None, match=None, recursive=False, max_length=100, **options)

一个 :class:`CharField` 内容只限于文件系统内特定目录下的文件名，有几个特殊参数, 其中第一个是
**必须的**:

.. attribute:: FilePathField.path

    必传。 :class:`FilePathField` 应该得到的选择目录的绝对文件系统路径。例如: ``"/home/images"``.

.. attribute:: FilePathField.match

    可选。 一个字符串形式的正则表达式。:class:`FilePathField` 使用其过滤文件名。
    regex将应用于基本文件名，而不是全部路径。例如：``"foo.*\.txt$"`` 将会匹配到
    ``foo23.txt`` 但不会匹配到 ``bar.txt`` 或 ``foo23.png`` 。

.. attribute:: FilePathField.recursive

    可选。 只能是 ``True`` 或者 ``False`` 。默认为 ``False`` 。声明是否包含所有子目录的 :attr:`~FilePathField.path` 。

.. attribute:: FilePathField.allow_files

    可选。 只能是 ``True`` 或者 ``False`` 。默认为  ``True`` 。 声明是否包含指定位置的文件
    该参数和
    :attr:`~FilePathField.allow_folders` 必须有一个为 ``True`` 。

.. attribute:: FilePathField.allow_folders

    可选。 只能是 ``True`` 或者 ``False`` 。默认为 ``False`` 。 声明是否包含指定位置的文件夹。 该参数
    和 :attr:`~FilePathField.allow_files` 必须有一个为 ``True`` 。

当然，这些参数可以同时使用。

有一点需要提醒的是 :attr:`~FilePathField.match` 只匹配基本文件名,
而不是整个文件路径,例如::

    FilePathField(path="/home/images", match="foo.*", recursive=True)

...将会匹配 ``/home/images/foo.png`` 但不会匹配 ``/home/images/foo/bar.png``
因为 :attr:`~FilePathField.match` 只作用于基本文件名
(``foo.png`` 和 ``bar.png``)。

:class:`FilePathField` 再数据库中以 ``varchar`` 类型存在
默认最大长度为 100 个字节。和普通一段一样，可以使用
:attr:`~CharField.max_length` 参数修改长度限制。

``FloatField``
--------------

.. class:: FloatField(**options)

用Python的一个 ``float`` 实例来表示一个浮点数。

如果属性 :attr:`~django.forms.Field.localize` 为 ``False`` 则默认表单部件是一个 :class:`~django.forms.NumberInput` ，
否则是一个
:class:`~django.forms.TextInput` 。

.. _floatfield_vs_decimalfield:

.. admonition:: ``FloatField`` vs. ``DecimalField``

    有时候 :class:`FloatField` 类可能会和
    :class:`DecimalField` 类产生混淆。虽然它们表示都表示实数，但是二者表示数字的方式不一样。
    ``FloatField`` 使用Python内部的 ``float``
    类型，而 ``DecimalField`` 使用的是Python的 ``Decimal`` 类型。
    想要了解更多二者的差别, 可以查看Python文档中的
    :mod:`decimal` 模块。

``ImageField``
--------------

.. class:: ImageField(upload_to=None, height_field=None, width_field=None, max_length=100, **options)

继承了:class:`FileField` 类的所有方法和属性，而且还对上传的对象进行校验，确保它是个有效的image。

除了从 :class:`FileField` 继承的属性外，
:class:`ImageField` 类还增加了 ``height`` 和 ``width`` 属性。

为了更便捷的去用这些属性值, :class:`ImageField` 有两个额外的可选参数:

.. attribute:: ImageField.height_field

    该属性的设定会在模型实例保存时,自动填充图片的高度。

.. attribute:: ImageField.width_field

    该属性的设定会在模型实例保存时,自动填充图片的宽度。

ImageField需要 `Pillow`_ 库支持。

.. _Pillow: https://pillow.readthedocs.io/en/latest/

:class:`ImageField` 实例在数据库中以 ``varchar`` 类型存在，默认最大长度为100个字节，
和普通字段一样。也可以使用 :attr:`~CharField.max_length` 参数限制。

该字段默认的表单部件是一个
:class:`~django.forms.ClearableFileInput` 。

``IntegerField``
----------------

.. class:: IntegerField(**options)

一个整数。在Django所有支持的数据库中，``-2147483648`` 到 ``2147483647`` 范围才是合法的。
如果设置了 :attr:`~django.forms.Field.localize` 为 ``False`` ,那么默认表单部件将是一个
:class:`~django.forms.NumberInput` 否则是一个 :class:`~django.forms.TextInput` 。

``GenericIPAddressField``
-------------------------

.. class:: GenericIPAddressField(protocol='both', unpack_ipv4=False, **options)

一条IPv4或IPv6地址, 字符串格式是 (e.g. ``192.0.2.30`` 或者
``2a02:42fe::4``)。 默认的表单部件是
:class:`~django.forms.TextInput`.

IPv6地址规范遵循 :rfc:`4291#section-2.2` 2.2 章节, 包括使用该部分第3段中建议的IPv4格式, 如
``::ffff:192.0.2.0`` 。 例如, ``2001:0::0:01`` 将被转换成
``2001::1``, ``::ffff:0a0a:0a0a`` 转换成 ``::ffff:10.10.10.10`` 。所有字符都转换为小写。

.. attribute:: GenericIPAddressField.protocol

    限制指定协议的有效输入。接受的值为
    ``'both'`` (默认), ``'IPv4'``
    或 ``'IPv6'`` 。匹配不区分大小写。

.. attribute:: GenericIPAddressField.unpack_ipv4

    解析IPv4映射地址如 ``::ffff:192.0.2.1``.
    如果启用该选项，则地址将被解析到
    ``192.0.2.1`` 。默认是禁用。只有当
    ``protocol`` 设置为 ``'both'`` 时才可以使用。

如果允许空值，则必须允许null值，因为空值存储为null。

``NullBooleanField``
--------------------

.. class:: NullBooleanField(**options)

类似 :class:`BooleanField`, 但是它允许 ``NULL`` 值。
可以用它来代替 :class:`BooleanField` 字段设置 ``null=True`` 的情况。它的默认表单是
:class:`~django.forms.NullBooleanSelect` 。

``PositiveIntegerField``
------------------------

.. class:: PositiveIntegerField(**options)

类似 :class:`IntegerField` 类型, 但必须是整数或者零 (``0``)。
取值范围是 ``0`` 到 ``2147483647`` 。
允许 ``0`` 是为了向后兼容。

``PositiveSmallIntegerField``
-----------------------------

.. class:: PositiveSmallIntegerField(**options)

类似 :class:`PositiveIntegerField` 类型， 但其允许的值必须小于某个特定值
(有数据库决定)。在Django所支持的数据库中 ``0`` 到 ``32767`` 内的值是绝对允许的。

``SlugField``
-------------

.. class:: SlugField(max_length=50, **options)

:term:`Slug` 是一个新闻术语， 通常叫做短标题。slug只能包含字母、数字、下划线或者是连字符。通常它们是用来放在URL里的。


类似 CharField 类型， 可以指定 :attr:`~CharField.max_length` (请参阅该部分中的有关数据库可移植性的说明和
:attr:`~CharField.max_length` )
如果没有指定 :attr:`~CharField.max_length` 属性，将默认使用50。

同时 :attr:`Field.db_index` 设置为 ``True`` 。

比较可行的做法是根据其他值的内容自动预填 SlugField 的值。
在admin中可以使用
:attr:`~django.contrib.admin.ModelAdmin.prepopulated_fields` 来实现。

.. attribute:: SlugField.allow_unicode

    .. versionadded:: 1.9

    如果设置为 ``True``, 该字段除了ASCII之外，还接受Unicode字母。默认为 ``False``.

``SmallIntegerField``
---------------------

.. class:: SmallIntegerField(**options)

类似 :class:`IntegerField`, 但只允许某些特定的值
(数据库决定)。 在Django所支持的数据库中 ``-32768`` 到 ``32767`` 之前是绝对允许的。

``TextField``
-------------

.. class:: TextField(**options)

大文本字段。默认的表单部件是一个
:class:`~django.forms.Textarea` 。

如果指定了 ``max_length`` 属性, 它将会在渲染页面时表单部件
:class:`~django.forms.Textarea` 中体现出来，但是却不会在模型和数据库中有这个限制。
如果需要这样清使用
:class:`CharField` 类型。

.. admonition:: MySQL 用户

    如果在 MySQLdb 1.2.1p2 中使用该字段，并且是 ``utf8_bin`` 排序规则(默认*不是* 这个)
    则需要注意几个问题。参考 :ref:`MySQL database notes <mysql-collation>` 。

``TimeField``
-------------

.. class:: TimeField(auto_now=False, auto_now_add=False, **options)

时间字段, 类似于Python ``datetime.time`` 实例. 和 :class:`DateField` 具有相同的选项.

默认的表单部件是一个 :class:`~django.forms.TextInput`.
在Admin中添加了一些JavaScript快捷方式。

``URLField``
------------

.. class:: URLField(max_length=200, **options)

一个 :class:`CharField` 类型的URL.

默认的表单部件是一个 :class:`~django.forms.TextInput`.

与所有 :class:`CharField` 子类一样, :class:`URLField` 接受
:attr:`~CharField.max_length` 可选参数. 如果没有特别指定
:attr:`~CharField.max_length` 默认长度是200.

``UUIDField``
-------------

.. class:: UUIDField(**options)

一个用于存储唯一标识符的字段. 使用 Python 的
:class:`~python:uuid.UUID` 类. 如果使用 PostgreSQL, 将使用
``uuid`` 数据类型储存, 其他情况是一个 ``char(32)``.

惟一的标识符是很好用来替代 :class:`AutoField` 类型
:attr:`~Field.primary_key` 的方法. 数据库不会自动生成UUID值, 所以最好使用 :attr:`~Field.default` 参数::

    import uuid
    from django.db import models

    class MyUUIDModel(models.Model):
        id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
        # other fields

注意传入的是一个可调用的对象(即一个省略括号的方法) 而不是传入一个 ``UUID`` 实例给 ``default`` .

关系字段
========

.. module:: django.db.models.fields.related
   :synopsis: Related field types

.. currentmodule:: django.db.models

Django 同样定义了一系列的字段来描述数据库之间的关联.

.. _ref-foreignkey:

``ForeignKey``
--------------

.. class:: ForeignKey(othermodel, on_delete, **options)

多对一关系. 需要一个位置参数: 与该模型关联的类.

.. versionchanged:: 1.9

    ``on_delete`` 现在可以用作第二个位置参数(之前它通常只是作为一个关键字参数传递).
    在Django 2.0中，这将是一个必传的参数。

.. _recursive-relationships:

若要创建一个递归的关联 —— 对象与自己具有多对一的关系 —— 请使用
``models.ForeignKey('self',
on_delete=models.CASCADE)``.

.. _lazy-relationships:

如果你需要关联到一个还没有定义的模型，你可以使用模型的名字而不用模型对象本身::

    from django.db import models

    class Car(models.Model):
        manufacturer = models.ForeignKey(
            'Manufacturer',
            on_delete=models.CASCADE,
        )
        # ...

    class Manufacturer(models.Model):
        # ...
        pass

在 :ref:`抽象模型 <abstract-base-classes>` 上以这种方式定义的关系在模型被作为具体模型进行子类化并且与抽象模型的 ``app_label`` 没有关联时:

.. snippet::
    :filename: products/models.py

    from django.db import models

    class AbstractCar(models.Model):
        manufacturer = models.ForeignKey('Manufacturer', on_delete=models.CASCADE)

        class Meta:
            abstract = True

.. snippet::
    :filename: production/models.py

    from django.db import models
    from products.models import AbstractCar

    class Manufacturer(models.Model):
        pass

    class Car(AbstractCar):
        pass

    # Car.manufacturer will point to `production.Manufacturer` here.

若要引用在其它应用中定义的模型，必须使用带有完整标签名的模型来显式指定。例如，如果上面提到的 ``Manufacturer`` 模型是在一个名为 ``production`` 的应用中定义的，就应该这样使用它::

    class Car(models.Model):
        manufacturer = models.ForeignKey(
            'production.Manufacturer',
            on_delete=models.CASCADE,
        )

当两个应用有相互导入情况时，这种引用将会很有帮助。

``ForeignKey`` 会自动创建数据库索引。如果你创建外键是为了一致性而不是用来Join，或者如果你将创建其它索引例如部分或多列索引，也许想要避免索引的开销。
这时可以通过设置 :attr:`~Field.db_index` 为 ``False`` 来取消。

数据库中表示
~~~~~~~~~~~~~

在背后，Django 会在字段名上添加 ``"_id"`` 来创建数据库中的列名。在上面的例子中，``Car`` 模型的数据库表将会拥有一个 ``manufacturer_id`` 列。
（也可以显式指定 :attr:`~Field.db_column` ）。但是，你的代码永远不应该处理数据库中的列名称，除非你需要编写自定义的SQL。你应该只关注你的模型对象中的字段名称。

.. _foreign-key-arguments:

参数
~~~~~~

:class:`ForeignKey` 接受一系列可选参数来标识关联关系的具体细节。

.. attribute:: ForeignKey.on_delete

    当一个 :class:`ForeignKey` 引用的对象被删除时, django将模拟 :attr:`on_delete` 参数指定的SQL约束行为。举个例子,
    如果你有一个可以为空的 :class:`ForeignKey` ，在其引用的对象被删除的时你想把这个 :class:`ForeignKey` 设置为空. 可以这样::

        user = models.ForeignKey(
            User,
            models.SET_NULL,
            blank=True,
            null=True,
        )

    .. deprecated:: 1.9

        :attr:`~ForeignKey.on_delete` 在Django 2.0 将成为一个必需的参数。在旧版本默值认为 ``CASCADE``.

:attr:`~ForeignKey.on_delete` 在
:mod:`django.db.models` 中可选的值有:

* .. attribute:: CASCADE

    级联删除。Django模拟了SQL约束在DELETE CASCADE上的行为，并删除ForeignKey的对象。

* .. attribute:: PROTECT

    抛出 :exc:`~django.db.models.ProtectedError` 异常以阻止被引用对象的删除, 它是
    :exc:`django.db.IntegrityError` 的一个子类.

* .. attribute:: SET_NULL

    重新设置 :class:`ForeignKey` 为null; 仅在 :attr:`~Field.null` 设置 ``True`` 时才能使用.

* .. attribute:: SET_DEFAULT

    将 :class:`ForeignKey` 设置为默认值; 这时必须设置 :class:`ForeignKey` 的默认值参数.

* .. function:: SET()

    设置 :class:`ForeignKey` 为 :func:`~django.db.models.SET()` 的接收值, 如果传递的是一个可调用对象，则为调用后的结果。
    在大部分情形下，传递一个可调用对象用于避免 models.py 在导入时执行查询::

        from django.conf import settings
        from django.contrib.auth import get_user_model
        from django.db import models

        def get_sentinel_user():
            return get_user_model().objects.get_or_create(username='deleted')[0]

        class MyModel(models.Model):
            user = models.ForeignKey(
                settings.AUTH_USER_MODEL,
                on_delete=models.SET(get_sentinel_user),
            )

* .. attribute:: DO_NOTHING

    不做任何动作. 如果删除关联数据时, 除非手动添加 ``ON DELETE`` 约束, 否则会引发 :exc:`~django.db.IntegrityError` 异常.

.. attribute:: ForeignKey.limit_choices_to

    当这个字段使用 ``ModelForm`` 或者Admin 渲染时（默认情况下，查询集中的所有对象都可以使用）, 为这个字段设置一个可用的选项。
    它可以是一个字典、一个 :class:`~django.db.models.Q` 对象或者一个返回字典或 :class:`~django.db.models.Q` 的可调用对象.

    例如::

        staff_member = models.ForeignKey(
            User,
            on_delete=models.CASCADE,
            limit_choices_to={'is_staff': True},
        )

    这样使在 ``ModelForm`` 对应的字段只列出 ``Users`` 的 ``is_staff=True``. 这在Django 的Admin 中也可能有用处。

    使用可调用对象的形式同样非常有用，比如和Python的 `datetime` 模块一起使用来限制选择的时间范围.
    例如::

        def limit_pub_date_choices():
            return {'pub_date__lte': datetime.date.utcnow()}

        limit_choices_to = limit_pub_date_choices

    如果 ``limit_choices_to`` 本身或者返回的是一个 :class:`Q object
    <django.db.models.Q>` 对象, 这对 :ref:`复合查询
    <complex-lookups-with-q>` 很有用. 当字段没有在模型的 ``ModelAdmin`` 中的 :attr:`~django.contrib.admin.ModelAdmin.raw_id_fields` 中列出时,
    它将只会影响Admin中的选项.

    .. note::

        如果 ``limit_choices_to`` 是可调用对象, 那么这个对象将在每次创建新表单实例的时候调用.
        它也可能在模型校验的时候调用, 例如被management命令或者Admin检验. Admin会多次构造查询集来验证表单在各种边缘情况下的输入,
        所以这个可调用对象可能会被调用多次.

.. attribute:: ForeignKey.related_name

    这个名称用于使关联的对象反查到源对象. 它还是 :attr:`related_query_name` 的默认值（关联的模型进行反向过滤时使用的名称）.
    完整的解释和示例参见 :ref:`关联对象文档 <backwards-related-objects>` .
    请注意，在 :ref:`抽象模型 <abstract-base-classes>` 定义关系时，必须设置此值；
    当您这样做时, 可以使用 :ref:`一些特殊语法 <abstract-related-name>` .

    如果你不想让Django创建反向关联, 可以设置 ``related_name`` 为 ``'+'`` 或者以 ``'+'`` 结尾. 例如,
    下面这行将确保 ``User`` 模型将不会有到这个模型的反向关联::

        user = models.ForeignKey(
            User,
            on_delete=models.CASCADE,
            related_name='+',
        )

.. attribute:: ForeignKey.related_query_name

    这个名称用于目标模型的反向过滤. 如果设置了 :attr:`related_name` 或者
    :attr:`~django.db.models.Options.default_related_name`, 将用它们作为默认值.
    否则默认值就是模型的名称::

        # Declare the ForeignKey with related_query_name
        class Tag(models.Model):
            article = models.ForeignKey(
                Article,
                on_delete=models.CASCADE,
                related_name="tags",
                related_query_name="tag",
            )
            name = models.CharField(max_length=255)

        # That's now the name of the reverse filter
        Article.objects.filter(tag__name="important")

    Like :attr:`related_name`, ``related_query_name`` supports app label and
    class interpolation via .
    像 :attr:`related_name` 一样, ``related_query_name`` 可以通过 :ref:`一些特殊语法 <abstract-related-name>` 来支持应用标签和类插值.

.. attribute:: ForeignKey.to_field

    关联到的关联对象的字段名称. 默认情况下, Django 使用关联对象的主键. 如果引用其他字段, 该字段必须具有 ``unique=True``.

.. attribute:: ForeignKey.db_constraint

    决定是否在数据库中为这个外键创建约束. 默认值为 ``True``, 这个符合大多数场景; 将此设置为 ``False`` 会对数据完整性非常不利.
    即便如此，有一些场景你也许需要这么设置：

    * 有遗留的无效数据.
    * 正在对数据库缩容.

    如果设置成 ``False``时，访问一个不存在的关联对象将抛出 ``DoesNotExist`` 异常.

.. attribute:: ForeignKey.swappable

    如果该 :class:`ForeignKey` 指向一个可切换swappable的模型, 该属性将决定迁移框架的行为.
    如果设置为 ``True`` -默认值- ，那么如果 :class:`ForeignKey` 指向的模型与 ``settings.AUTH_USER_MODEL`` 匹配（或其它可切换的模型），
    该关联关系会被保存在迁移migration中，且使用的是对setting 中引用而不是直接对模型的引用。

    只有当你的模型想永远指向切换后的模型 —— 例如如果它是专门为你的自定义用户模型设计的模型时，这时就可以将它设置成 ``False``.

    设置为 ``False`` 并不表示你可以引用可切换的模型,即使在它被切换出去之后 —— ``False`` 只是表示生成的迁移中ForeignKey 将始终引用你指定的准确模型
    （所以，如果用户试图允许一个你不支持的User 模型时将会失败）。

    如果对此有疑问，请保留它的默认值 ``True``.

``ManyToManyField``
-------------------

.. class:: ManyToManyField(othermodel, **options)

一个多对多关联关系. 必须传入一个关键字参数：与该模型关联的类，它与 :class:`ForeignKey` 的完全一样，包括
:ref:`recursive <recursive-relationships>` 和
:ref:`lazy <lazy-relationships>` 关系类型.

可以通过字段的 :class:`~django.db.models.fields.related.RelatedManager` 进行关联对象的添加/删除和创建.

数据库表示
~~~~~~~~~~~~

在数据库层, Django 通过创建一个中间表来表示多对多关系.
默认情况下, 这张中间表的名称是由使用多对多字段的名称和包含这张表的模型的名称组合产生.
由于某些数据库表名字支持的长度有限, 这种情况下, 表的名称将自动截短到64个字符并加上一个唯一哈希值.
这意味着你可能会看到像 ``author_books_9cdf4`` 这样的表名；这是没有问题的. 你可以使用
:attr:`~ManyToManyField.db_table` 选项手动提供中间表的名称.

.. _manytomany-arguments:

参数
~~~~~~~

:class:`ManyToManyField` 接收一个参数集来控制关联关系功能 -- 所有选项 -- 都是可选的.

.. attribute:: ManyToManyField.related_name

    和 :attr:`ForeignKey.related_name` 相同.

.. attribute:: ManyToManyField.related_query_name

    和 :attr:`ForeignKey.related_query_name` 相同.

.. attribute:: ManyToManyField.limit_choices_to

    和 :attr:`ForeignKey.limit_choices_to` 相同.

    如果 ``ManyToManyField`` 使用
    :attr:`~ManyToManyField.through` 自定义中间表时, ``limit_choices_to`` 无效.

.. attribute:: ManyToManyField.symmetrical

    只用于与自身进行关联的ManyToManyField. 例如下面的模型::

        from django.db import models

        class Person(models.Model):
            friends = models.ManyToManyField("self")


    当Django 处理这个模型的时候, 它定义了该模型具有一个与自身具有多对多关联的 :class:`ManyToManyField`.
    因此它不会向 ``Person`` 类添加 ``person_set`` 属性. Django会假定这个 :class:`ManyToManyField` 字段是对称的
    —— 即, 如果我是你的朋友，那么你也是我的朋友.

    如果你希望与 ``self`` 进行多对多关联的关系是非对称的，可以设置
    :attr:`~ManyToManyField.symmetrical` 为 ``False``.
    这会强制让Django给反向的关联关系添加一个描述器, 以使得 :class:`ManyToManyField` 的关联关系不是对称的.

.. attribute:: ManyToManyField.through

    Django会自动创建一个表来管理多对多关系. 不过, 如果你希望手动指定这个中间表, 可以使用
    :attr:`~ManyToManyField.through` 选项来告诉Django模型你想使用的中间表.

    这个选项最常见的使用场景是用来 :ref:`关联更多的数据
    <intermediary-manytomany>`.

    如果没有显式指定 ``through`` 的模型, 仍然会有一个隐式的 ``through`` 模型类，
    你可以用它来直接访问对应的表示关联关系的数据表. 它由三个字段来连接模型。

    如果源模型和目标模型不相同, 则生成以下字段:

    * ``id``: 关系的主键.
    * ``<containing_model>_id``: 声明 ``ManyToManyField`` 的模型的 ``id``.
    * ``<other_model>_id``: 被 ``ManyToManyField`` 选项指向的模型的 ``id``.

    如果 ``ManyToManyField`` 选项的源模型和目标模型是同一个, 则生成以下字段:

    * ``id``: 关系的主键.
    * ``from_<model>_id``: 源模型实例的 ``id``.
    * ``to_<model>_id``: 目标模型实例的 ``id``.

    这个类可以让一个给定的模型像普通的模型那样查询与之相关联的记录。

.. attribute:: ManyToManyField.through_fields

    只能在自定义中间模型的时候使用. 通常情况下Django会自行决定使用中间模型的哪些字段来建立多对多关联.
    但是，在如下模型中::

        from django.db import models

        class Person(models.Model):
            name = models.CharField(max_length=50)

        class Group(models.Model):
            name = models.CharField(max_length=128)
            members = models.ManyToManyField(
                Person,
                through='Membership',
                through_fields=('group', 'person'),
            )

        class Membership(models.Model):
            group = models.ForeignKey(Group, on_delete=models.CASCADE)
            person = models.ForeignKey(Person, on_delete=models.CASCADE)
            inviter = models.ForeignKey(
                Person,
                on_delete=models.CASCADE,
                related_name="membership_invites",
            )
            invite_reason = models.CharField(max_length=64)

    ``Membership`` 有 *两个* 到 ``Person`` 到外键(``person`` 和
    ``inviter``), 这样会导致关系不清晰, Django不知道使用哪一个外键。 在这种情况下,
    必须使用 ``through_fields`` 明确指定Django应该使用哪些外键，就像上面例子一样.

    ``through_fields`` 接受一个二元元组 ``('field1', 'field2')``,
    其中 ``field1`` 是指向定义了 :class:`ManyToManyField` 的模型的外键名字(本例中是 ``group``),
    ``field2`` 就是目标模型的外键的名 (本例中是 ``person``).

    当中间模型具有多个外键指向多对多关联关系模型中的任何一个（或两个）时, 就 *必须* 指定 ``through_fields``.
    当用到中间模型有多个外键指向该模型时，而你想显式指定Django应该用到的两个字段, 这同样适用于
    :ref:`递归关联关系 <recursive-relationships>`.

    递归的关联关系使用的中间模型始终定义为非对称的 -- 也就是 :attr:`symmetrical=False <ManyToManyField.symmetrical>`
    -- 所以具有源和目标的概念. 这种情况下，``'field1'`` 将作为关系的源，而 ``'field2'`` 作为目标.


.. attribute:: ManyToManyField.db_table

    用于存储多对多关系数据而创建的表的名称. 如果没有设置, Django将基于定义关联关系的模型和字段设置一个默认名称.

.. attribute:: ManyToManyField.db_constraint

    决定是否为中间表中的外键创建约束. 默认值 ``True``，这适合大多数场景；
    如将此设置为 ``False`` 可能对数据完整性非常友好. 即便如此，有一些场景也许需要这么设置：

    * 存在遗留的无效数据.
    * 正要对数据库缩容.

    不可以同时设置 ``db_constraint`` 和 ``through``.

.. attribute:: ManyToManyField.swappable

    如果该 :class:`ManyToManyField` 指向一个可切换swappable的模型, 该属性将决定迁移框架的行为.
    如果设置为 ``True``, -默认值- ，那么如果 :class:`ManyToManyField` 指向的模型与
    ``settings.AUTH_USER_MODEL`` 匹配（或其它可切换的模型）,
    该关联关系会被保存在迁移migration中，且使用的是对setting 中引用而不是直接对模型的引用.

    如果对此有疑问，请保留它的默认值 ``True``.

:class:`ManyToManyField` 不支持 :attr:`~Field.validators`.

:attr:`~Field.null` 不起任何作用，因为在数据库层不需要关系.

``OneToOneField``
-----------------

.. class:: OneToOneField(othermodel, on_delete, parent_link=False, **options)

一对一关联关系. 在概念上, 它类似于设置了 :attr:`unique=True <Field.unique>`
的 :class:`ForeignKey`, 但是，一对一关系的“reverse”部分将直接返回单个对象.

.. versionchanged:: 1.9

    ``on_delete`` 现在可以用作第二个位置参数 (以前它通常只作为关键字参数传递).
     在Django 2.0中，这将是一个必传的参数.

这是最有效的一种方式,即作为一个模型的主键,在某种程度上又“扩展”了另一个模型;
例如，:ref:`multi-table-inheritance` 就是通过将一个隐式一对一关系从子模型添加到父模型来实现的.

需要一个位置参数：与该模型关联的类. 它的工作方式与 :class:`ForeignKey` 完全一致,
包括所有与 :ref:`recursive <recursive-relationships>` 和 :ref:`lazy <lazy-relationships>` 相关的选项.

如果没有指定 ``OneToOneField`` 的 :attr:`~ForeignKey.related_name` 参数，Django将使用当前模型的小写的名称作为默认值.

例如下面例子::

    from django.conf import settings
    from django.db import models

    class MySpecialUser(models.Model):
        user = models.OneToOneField(
            settings.AUTH_USER_MODEL,
            on_delete=models.CASCADE,
        )
        supervisor = models.OneToOneField(
            settings.AUTH_USER_MODEL,
            on_delete=models.CASCADE,
            related_name='supervisor_of',
        )

这将使 ``User`` 模型具有以下属性::

    >>> user = User.objects.get(pk=1)
    >>> hasattr(user, 'myspecialuser')
    True
    >>> hasattr(user, 'supervisor_of')
    True

反向访问关联关系时, 如果关联的对象不存在对应的实例, 将抛出 ``DoesNotExist`` 异常.
例如, 如果一个 ``User`` 没有 ``MySpecialUser`` 指定的supervisor::

    >>> user.supervisor_of
    Traceback (most recent call last):
        ...
    DoesNotExist: User matching query does not exist.

.. _onetoone-arguments:

除此之外, ``OneToOneField`` 除了接收 :class:`ForeignKey` 接收的所有额外的参数之外, 还有另外一个参数:

.. attribute:: OneToOneField.parent_link

    当它为 ``True`` 并在继承自另一个 :term:`concrete model` 的模型中使用时，
    表示该字段应该用于反查的父类的链接，而不是在子类化时隐式创建的 ``OneToOneField``.

``OneToOneField`` 的使用示例参见 :doc:`One-to-one关联关系 </topics/db/examples/one_to_one>`.

Field API参考
================

.. class:: Field

    ``Field`` 是一个抽象的类, 用来表示数据库中的表的列.

    Django使用字段创建数据库表(:meth:`db_type`), 将Python类型映射到数据库(:meth:`get_prep_value`),
    反之亦然(:meth:`from_db_value`).

    在各个版本的Django API中field是最根本的部分, 尤其是
    :class:`models <django.db.models.Model>` 和
    :class:`querysets <django.db.models.query.QuerySet>`.

    在模型中, 字段被实例化为类的属性, 并表现为一个特定的表的列，
    详情参考 :doc:`/topics/db/models`.
    它具有许多属性，如 :attr:`null` 和 :attr:`unique` 等, 以及用于将字段值映射到数据库特定值的方法.

    ``Field`` 是 :class:`~django.db.models.lookups.RegisterLookupMixin` 的子类，
    因此可以在其上注册 :class:`~django.db.models.Transform` 和
    :class:`~django.db.models.Lookup`，也就可以在 ``QuerySet`` 中使用(例如 ``field_name__="foo"``).
    所有 :ref:`内置查询 <field-lookups>` 都是默认注册好的.

    Django的所有内建字段，如 :class:`CharField` 都是 ``Field`` 的特殊实现.
    如果需要自定义字段，则可以将任何内置字段重载，也可以重写一个新的 ``Field``.
    无论哪种情况，请参考 :doc:`/howto/custom-model-fields`.

    .. attribute:: description

        字段的详细描述，例如用于 :mod:`django.contrib.admindocs` 应用程序.

        描述可以是以下形式::

            description = _("String (up to %(max_length)s)")

        其中参数从字段的 ``__dict__`` 去获取.

    为了将 ``Field`` 映​​射到数据库特定类型，Django开放了几种方法:

    .. method:: get_internal_type()

        返回一个命名此字段的字符串用于后端特定用途. 默认情况下, 它返回类名.

        有关自定义字段中的用法，请参见 :ref:`emulating-built-in-field-types`.

    .. method:: db_type(connection)

        返回 :class:`Field` 的数据库列数据类型，同时考虑 ``connection``.

        有关自定义字段中使用 :ref:`custom-database-types`.

    .. method:: rel_db_type(connection)

        .. versionadded:: 1.10

        返回指向 :class:`Field` 的字段的数据库列数据类型，例如 ``ForeignKey`` 和 ``OneToOneField``,
        并考虑 ``connection``.

        有关自定义字段中使用 :ref:`custom-database-types`.

    有三种主要情况，Django需要与数据库后端和字段交互:

    * 当它查询数据库 (Python value -> database backend value)
    * 当它从数据库加载数据 (database backend value -> Python
      value)
    * 当它保存到数据库 (Python value -> database backend value)

    查询时, :meth:`get_db_prep_value` 和 :meth:`get_prep_value` 将被用到:

    .. method:: get_prep_value(value)

        ``value`` 是模型属性的当前值, 该方法应返回用作查询的参数的格式数据.

        有关使用方法，请参阅 :ref:`converting-python-objects-to-query-values`.

    .. method:: get_db_prep_value(value, connection, prepared=False)

        将 ``value`` 转换为特定后端值. 如果设置 ``prepared=True``，则默认返回 ``value``.
        如果为 ``False`` 则返回 :meth:`~Field.get_prep_value`.

        有关使用方法，请参阅 :ref:`converting-query-values-to-database-values`.

    加载数据时 :meth:`from_db_value` 将被使用:

    .. method:: from_db_value(value, expression, connection, context)

        将数据库返回的值转换为Python对象. 它与 :meth:`get_prep_value` 相反.

        此方法不使用于大多数内置字段, 因为这些字段在数据库后端已返回正确的Python类型，或后端本身执行转换.

        有关使用方法，请参阅  :ref:`converting-values-to-python-objects`.

        .. note::

            出于性能原因，``from_db_value`` 没有在不需要它的字段上实现(所有Django字段).
            因此，您不能在定义中调用 ``super``.

    保存数据时, :meth:`pre_save` 和 :meth:`get_db_prep_save` 将被使用:

    .. method:: get_db_prep_save(value, connection)

        类似于 :meth:`get_db_prep_value`, 但仅仅在字段值一定会 *保存* 到数据库时调用. 默认返回
        :meth:`get_db_prep_value`.

    .. method:: pre_save(model_instance, add)

        在 ``get_db_prep_save()`` 之前调用, 以在保存之前准备好值. (e.g. for :attr:`DateField.auto_now`).

        ``model_instance`` 是该字段所属的实例， ``add`` 是首次将实例保存到数据库.

        它会返回此字段的 ``model_instance`` 适当属性的值. 属性名称位于 ``self.attname``
        (这个由 :class:`~django.db.models.Field` 设置)

        有关使用方法，请参阅 :ref:`preprocessing-values-before-saving`.

    字段通常以不同的类型接收它们的值，无论是序列化还是表单.

    .. method:: to_python(value)

        将value值转换为正确的python对象. 它作为 :meth:`value_to_string` 的反向操作，也是在
        :meth:`~django.db.models.Model.clean` 中调用.

        有关使用方法，请参阅 :ref:`converting-values-to-python-objects`.

    除了保存到数据库，字段还需要知道如何序列化值:

    .. method:: value_to_string(obj)

        将 ``obj`` 转换成字符串. 用于序列化字段的值.

        有关使用方法，请参阅  :ref:`converting-model-field-to-serialization`.

    使用 :class:`model forms <django.forms.ModelForm>` 时, ``Field``
    需要知道应由哪个表单字段表示:

    .. method:: formfield(form_class=None, choices_form_class=None, **kwargs)

        返回 :class:`~django.forms.ModelForm` 该字段的默认 :class:`django.forms.Field`.

        默认情况下, 如果 ``form_class`` 和 ``choices_form_class`` 都是 ``None``,
        则使用 :class:`~django.forms.CharField`. 如果字段没有指定
        :attr:`~django.db.models.Field.choices` 和 ``choices_form_class``,
        则使用 :class:`~django.forms.TypedChoiceField`.

        有关用法, 参见 :ref:`specifying-form-field-for-model-field`.

    .. method:: deconstruct()

        返回足够多信息的4元元祖, 用来重新创建字段:

        1. 模型中的字段名称.
        2. 字段的导入路径 (e.g. ``"django.db.models.IntegerField"``).
            这最好是能够兼容的版本，所以不要太具体更好.
        3. 位置参数列表.
        4. 关键词参数列表.

        必须将此方法添加到1.7之前的字段, 才能使用 :doc:`/topics/migrations` 迁移其数据.

.. _model-field-attributes:

===============
字段属性参考
===============

每个 ``Field`` 实例包含几个允许内省其行为的属性. 当需要编写依赖于字段功能的代码时，就可以使用这些属性而不是 ``isinstance`` 检查.
这些属性可以与 :ref:`Model._meta API
<model-meta-field-api>` 一起使用, 缩小搜索特定字段类型的范围.自定义模型字段应实现这些标志.

字段属性
=========

.. attribute:: Field.auto_created

    布尔标志, 指示是否自动创建字段, 例如模型继承使用的 ``OneToOneField``.

.. attribute:: Field.concrete

    布尔标志, 指示字段是否具有与其相关联的数据库列.

.. attribute:: Field.hidden

    布尔标志, 指示字段是否用于支持另一个非隐藏字段的功能. (e.g. ``content_type`` 和 ``object_id`` 构成了字段 ``GenericForeignKey``).
    ``hidden`` 标志用于区分模型上的公共字段子集构成模型上的所有字段.

    .. note::

        :meth:`Options.get_fields()
        <django.db.models.options.Options.get_fields()>`
        默认不返回隐藏字段. 可以通过设置 ``include_hidden=True`` 返回隐藏字段.

.. attribute:: Field.is_relation

    布尔标志，表示字段是否包含对一个或多个其他模型的引用. (e.g. ``ForeignKey``,
    ``ManyToManyField``, ``OneToOneField``, etc.).

.. attribute:: Field.model

    返回定义字段的模型。如果在模型的超类上定义了一个字段，``model`` 将引用父类，而不是实例的类.

关系字段属性
=============

这些属性用于查询关系的基数和其他详细信息. 这些属性存在于所有字段中; 但是, 如果该字段是关系类型 ( :attr:`Field.is_relation=True <Field.is_relation>`)
它们将只有布尔值 (而不是 ``None``).

.. attribute:: Field.many_to_many

    布尔标志, 如果该字段具有多对多关系，则为 ``True``; 否则 ``False``. Django中只有 ``ManyToManyField`` 为 ``True``.

.. attribute:: Field.many_to_one

    布尔标志, 如果字段具有多对一关联关系则为 ``True``. 比如 ``ForeignKey``; 其他为 ``False``.

.. attribute:: Field.one_to_many

    布尔标志, 如果字段具有一对多关联关系则为 ``True`` . 比如
    ``GenericRelation`` 或者反向 ``ForeignKey``; 其他情况为 ``False``.

.. attribute:: Field.one_to_one

    布尔标志, 如果字段具有一对一关联关系则为 ``True`` .比如
    ``OneToOneField``; 其他情况为 ``False``.

.. attribute:: Field.related_model

    指向字段关联的模型. 例如,
    ``ForeignKey(Author, on_delete=models.CASCADE)`` 中的 ``Author`` .
    ``GenericForeignKey`` 中的 ``related_model`` 始终为 ``None``.
