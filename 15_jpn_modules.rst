.. _man-modules:

.. 
*********
 Modules
*********

*********
 モジュール
*********

.. index:: module, baremodule, using, import, export, importall

.. 
 Modules in Julia are separate variable workspaces, i.e. they introduce
 a new global scope. They are delimited syntactically, inside ``module
 Name ... end``. Modules allow you to create top-level definitions (aka
 global variables) without worrying about name conflicts when your code
 is used together with somebody else's. Within a module, you can
 control which names from other modules are visible (via importing),
 and specify which of your names are intended to be public (via
 exporting).

Juliaのモジュールは区別された変数ワークスペースです。例えば、新しいグローバルスコープを導入します。
それらは、 ``module Name ... end`` の中で構文的に区切られます。モジュールは、
コードが他の箇所で一緒に使用されている場合でも、名前の競合を心配することなく、
トップレベルの定義（別名グローバル変数）を作成することを可能にします。モジュール内では、
他のモジュールのどの名前が（インポートによって）表示されるかを制御したり、
どの名前を（エクスポートによって）公開するかを指定することができます。

.. 
 The following example demonstrates the major features of modules. It is
 not meant to be run, but is shown for illustrative purposes::

次の例は、モジュールの主な機能を示しています。これは実行されるためではなく、説明の目的で示されています。::

    module MyModule
    using Lib

    using BigLib: thing1, thing2

    import Base.show

    importall OtherLib

    export MyType, foo

    type MyType
        x
    end

    bar(x) = 2x
    foo(a::MyType) = bar(a.x) + 1

    show(io::IO, a::MyType) = print(io, "MyType $(a.x)")
    end

.. 
 Note that the style is not to indent the body of the module, since
 that would typically lead to whole files being indented.

通常ファイル全体がインデントされるため、コードではモジュールのボディ部を字下げしないことに注意してください。

.. 
 This module defines a type ``MyType``, and two functions. Function
 ``foo`` and type ``MyType`` are exported, and so will be available for
 importing into other modules.  Function ``bar`` is private to
 ``MyModule``.

このモジュールは型 ``MyType`` および2つの関数を定義します。関数 ``foo`` と型 ``MyType`` がエクスポートされるため、
他のモジュールにインポートすることもできます。ファンクション ``bar`` は ``MyModule`` 専用です。

.. 
 The statement ``using Lib`` means that a module called ``Lib`` will be
 available for resolving names as needed. When a global variable is
 encountered that has no definition in the current module, the system
 will search for it among variables exported by ``Lib`` and import it if
 it is found there.
 This means that all uses of that global within the current module will
 resolve to the definition of that variable in ``Lib``.

``using Lib`` ステートメントは、 ``Lib`` モジュールは必要な場合に名前解決が可能であることを意味します。
グローバル変数が現在のモジュールに定義されていない場合、システムは ``Lib`` によりエクスポートされた変数から検索し、
もし一致するものがあればそれをインポートします。これは、現在のモジュール内のグローバルの全ての利用が、
``Lib`` 内の変数の定義により解決されることを意味します。

.. 
 The statement ``using BigLib: thing1, thing2`` is a syntactic shortcut for
 ``using BigLib.thing1, BigLib.thing2``.

ステートメント ``using BigLib: thing1, thing2`` は、 ``using BigLib.thing1, BigLib.thing2`` の構文上のショートカットです。

.. 
 The ``import`` keyword supports all the same syntax as ``using``, but only
 operates on a single name at a time. It does not add modules to be searched
 the way ``using`` does. ``import`` also differs from ``using`` in that
 functions must be imported using ``import`` to be extended with new methods.

``import`` キーワードは、 ``using`` と同じ構文をサポートしますが、一度に1つの名前でしか動作しません。
``import`` は、 ``using`` が行うような方法では、検索されるモジュールを追加することはしません。
``import`` は、新しいメソッドで拡張するために ``import`` を使用してインポートする必要がある点で、
``using`` とは異なります。

.. 
 In ``MyModule`` above we wanted to add a method to the standard ``show``
 function, so we had to write ``import Base.show``.
 Functions whose names are only visible via ``using`` cannot be extended.

上記の ``MyModule`` では、標準の ``show`` 関数にメソッドを追加したいため、
``import Base.show`` を書く必要がありました。 ``using`` を使用することでのみ名前がわかる関数は、
拡張することができません。

.. 
 The keyword ``importall`` explicitly imports all names exported by the
 specified module, as if ``import`` were individually used on all of them.

``importall`` キーワードは、個々に対して ``importall`` が使われたかのように、
指定されたモジュールによってエクスポートされた全ての名前を明示的にインポートします。

.. 
 Once a variable is made visible via ``using`` or ``import``, a module may
 not create its own variable with the same name.
 Imported variables are read-only; assigning to a global variable always
 affects a variable owned by the current module, or else raises an error.

変数を ``using`` または ``import`` で表示すると、モジュールは同じ名前の変数を作成しないことがあります。
インポートされた変数は読み取り専用です。グローバル変数の割り当ては、常に現在のモジュールが所有する変数に影響しますし、
そうでない場合は、エラーが発生します。


.. 
Summary of module usage
^^^^^^^^^^^^^^^^^^^^^^^

モジュールの使用のサマリ
^^^^^^^^^^^^^^^^^^^^^^^

.. 
 To load a module, two main keywords can be used: ``using`` and ``import``. To understand their differences, consider the following example::

モジュールをロードするには、 ``using`` と ``import`` の2つの主要なキーワードを使用できます。それらの違いを理解するには、
次の例を考えてみてください。::

    module MyModule

    export x, y

    x() = "x"
    y() = "y"
    p() = "p"

    end

.. 
 In this module we export the ``x`` and ``y`` functions (with the keyword ``export``), and also have the non-exported function ``p``. There are several different ways to load the Module and its inner functions into the current workspace:

このモジュールは、 ``x`` および ``y`` 関数（ ``export`` キーワードを使用）をエクスポートし、
エクスポートされていない関数 ``p`` も持ちます。モジュールとその内部関数を現在のワークスペースにロードするには、
いくつかの方法があります。:

..
 +------------------------------------+-----------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
 |Import Command                      | What is brought into scope                                                                    | Available for method extension                                         |
 +====================================+===============================================================================================+========================================================================+
 | ``using MyModule``                 | All ``export``\ ed names (``x`` and ``y``), ``MyModule.x``, ``MyModule.y`` and ``MyModule.p`` | ``MyModule.x``, ``MyModule.y`` and ``MyModule.p``                      |
 +------------------------------------+-----------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
 | ``using MyModule.x, MyModule.p``   | ``x`` and ``p``                                                                               |                                                                        |
 +------------------------------------+-----------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
 | ``using MyModule: x, p``           | ``x`` and ``p``                                                                               |                                                                        |
 +------------------------------------+-----------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
 | ``import MyModule``                | ``MyModule.x``, ``MyModule.y`` and ``MyModule.p``                                             | ``MyModule.x``, ``MyModule.y`` and ``MyModule.p``                      |
 +------------------------------------+-----------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
 | ``import MyModule.x, MyModule.p``  | ``x`` and ``p``                                                                               | ``x`` and ``p``                                                        |
 +------------------------------------+-----------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
 | ``import MyModule: x, p``          | ``x`` and ``p``                                                                               | ``x`` and ``p``                                                        |
 +------------------------------------+-----------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
 | ``importall MyModule``             |  All ``export``\ ed names (``x`` and ``y``)                                                   | ``x`` and ``y``                                                        |
 +------------------------------------+-----------------------------------------------------------------------------------------------+------------------------------------------------------------------------+


+------------------------------------+---------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
|インポートコマンド                     | スコープにインポートされるもの                                                                               | 可能なメソッド拡張                                                        |
+====================================+=========================================================================================================+========================================================================+
| ``using MyModule``                 | 全ての ``export``\ された名前 （ ``x`` および ``y`` ）, ``MyModule.x``, ``MyModule.y`` および ``MyModule.p`` | ``MyModule.x``, ``MyModule.y`` および ``MyModule.p``                    |
+------------------------------------+---------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
| ``using MyModule.x, MyModule.p``   | ``x`` および ``p``                                                                                       |                                                                        |
+------------------------------------+---------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
| ``using MyModule: x, p``           | ``x`` および ``p``                                                                                       |                                                                        |
+------------------------------------+---------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
| ``import MyModule``                | ``MyModule.x``, ``MyModule.y`` および ``MyModule.p``                                                     | ``MyModule.x``, ``MyModule.y`` および ``MyModule.p``                    |
+------------------------------------+---------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
| ``import MyModule.x, MyModule.p``  | ``x`` および ``p``                                                                                       | ``x`` および ``p``                                                      |
+------------------------------------+---------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
| ``import MyModule: x, p``          | ``x`` および ``p``                                                                                       | ``x`` および ``p``                                                      |
+------------------------------------+---------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------+
| ``importall MyModule``             |  全ての ``export``\ された名前 （ ``x`` および ``y`` ）                                                     | ``x`` および ``y``                                                      |
+------------------------------------+---------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------+

.. 
 Modules and files
 -----------------

モジュールとファイル
-----------------

.. 
 Files and file names are mostly unrelated to modules; modules are associated
 only with module expressions.
 One can have multiple files per module, and multiple modules per file::

ファイルとファイル名はほとんどがモジュールとは無関係です。モジュールはモジュール式とのみ関連付けられます。
1モジュールに対し複数のファイルを持つことができ、1ファイルに対し複数のモジュールを持つことができます。::

    module Foo

    include("file1.jl")
    include("file2.jl")

    end

.. 
 Including the same code in different modules provides mixin-like behavior.
 One could use this to run the same code with different base definitions,
 for example testing code by running it with "safe" versions of some
 operators::

異なるモジュールに同じコードを組み込むことで、mixinのような動作が実現します。これを使用することで、
異なる基本定義を持つ同じコードを実行することができます。例えば、「安全な」演算子のバージョンでコードを
実行してテストをすることができます。::

    module Normal
    include("mycode.jl")
    end

    module Testing
    include("safe_operators.jl")
    include("mycode.jl")
    end


.. 
 Standard modules
 ----------------

標準モジュール
----------------

.. 
 There are three important standard modules: Main, Core, and Base.

モジュールには、Main、Core、Baseの3つの主要モジュールがあります。

.. 
 Main is the top-level module, and Julia starts with Main set as the
 current module.  Variables defined at the prompt go in Main, and
 ``whos()`` lists variables in Main.

Mainは最上位のモジュールであり、JuliaはカレントのモジュールとしてMainを設定しています。
プロンプトで定義された変数はMainに格納され、 ``whos()`` はMainの変数をリスト化します。

.. 
 Core contains all identifiers considered "built in" to the language, i.e.
 part of the core language and not libraries. Every module implicitly
 specifies ``using Core``, since you can't do anything without those
 definitions.

Coreには、使用している言語に「ビルトイン」とみなされる識別子、例えばライブラリではなくコア言語の一部が格納されます。
これらの定義なしには処理できないため、全てのモジュールは暗黙的に ``using Core`` を定義します。

.. 
 Base is the standard library (the contents of base/). All modules implicitly
 contain ``using Base``, since this is needed in the vast majority of cases.

Baseは標準ライブラリ（base/の内容）です。多くの場合に必要となるため、全てのモジュールは、暗黙的に ``using Base`` を持ちます。


.. 
 Default top-level definitions and bare modules
 ----------------------------------------------

デフォルトの最上位定義とベアモジュール
----------------------------------------------

.. 
 In addition to ``using Base``, modules also automatically contain a definition
 of the ``eval`` function, which evaluates expressions within the context of that module.

``using Base`` に加え、モジュールは、コンテキスト内の式を評価する ``eval`` 関数の定義を自動的に持ちます。

.. 
 If these default definitions are not wanted, modules can be defined using the
 keyword ``baremodule`` instead (note: ``Core`` is still imported, as per above).
 In terms of ``baremodule``, a standard ``module`` looks like this::

これらのデフォルト定義が望ましくない場合は、代わりにキーワード ``baremodule`` を使用してモジュールを定義することができます
（上記のように ``Core`` はインポートされます）。 ``baremodule`` に関して、標準モジュールは次のようになります。::

    baremodule Mod

    using Base

    eval(x) = Core.eval(Mod, x)
    eval(m,x) = Core.eval(m, x)

    ...

    end


.. 
 Relative and absolute module paths
 ----------------------------------

相対モジュールパスと絶対モジュールパス
----------------------------------

.. 
 Given the statement ``using Foo``, the system looks for ``Foo``
 within ``Main``. If the module does not exist, the system
 attempts to ``require("Foo")``, which typically results in loading
 code from an installed package.

``using Foo`` のステートメントが与えられた場合、システムは ``Main`` 内の ``Foo`` を検索します。
モジュールが存在しない場合、システムは ``require("Foo")`` を試み、
これはインストールされたパッケージからコードをロードします。

.. 
 However, some modules contain submodules, which means you sometimes
 need to access a module that is not directly available in ``Main``.
 There are two ways to do this. The first is to use an absolute path,
 for example ``using Base.Sort``. The second is to use a relative path,
 which makes it easier to import submodules of the current module or
 any of its enclosing modules::

しかし、モジュールの中にはサブモジュールが含まれているものがあり、
これは ``Main`` で直接利用できないモジュールにアクセスする必要がある場合があります。
これを行うには2つの方法があります。1つ目は ``using Base.Sort`` などの絶対パスを使用する方法です。
2つ目は、カレントモジュールまたはそのモジュールを含むサブモジュールのインポートを容易にする相対パスを使用する方法です。::

    module Parent

    module Utils
    ...
    end

    using .Utils

    ...
    end

.. 
 Here module ``Parent`` contains a submodule ``Utils``, and code in
 ``Parent`` wants the contents of ``Utils`` to be visible. This is
 done by starting the ``using`` path with a period. Adding more leading
 periods moves up additional levels in the module hierarchy. For example
 ``using ..Utils`` would look for ``Utils`` in ``Parent``'s enclosing
 module rather than in ``Parent`` itself.
 
ここではモジュール ``Parent`` にサブモジュール ``Utils`` が含まれ、
``Utils`` の内容は ``Parent`` のコードに対し表示される必要があります。
これは、 ``using`` パスをピリオドで開始することで行われます。先頭のピリオドを追加すると、
モジュール階層の追加レベルが上がります。例えば ``using ..Utils`` は、
``Parent`` 自体ではなく ``Parent`` に含まれるモジュール内で ``Utils`` を検索します。 

.. 
 Note that relative-import qualifiers are only valid in ``using`` and
 ``import`` statements.

相対インポート修飾子は、 ``using`` および ``import`` ステートメントでのみ有効であることに注意してください。

.. 
 Module file paths
 -----------------

モジュールファイルパス
-----------------

.. 
 The global variable LOAD_PATH contains the directories Julia searches for
 modules when calling ``require``. It can be extended using ``push!``::

グローバル変数LOAD_PATHには、 ``require`` を呼び出すときにJuliaがモジュールを検索するディレクトリが含まれています。
これは ``push!`` を使うことで拡張できます。::

    push!(LOAD_PATH, "/Path/To/My/Module/")

.. 
 Putting this statement in the file ``~/.juliarc.jl`` will extend LOAD_PATH
 on every Julia startup. Alternatively, the module load path can be
 extended by defining the environment variable JULIA_LOAD_PATH.

このステートメントを ``~/.juliarc.jl`` 内に記述すると、Juliaの起動時にLOAD_PATHを拡張します。
または、環境変数JULIA_LOAD_PATHを定義することでモジュールのロードパスを拡張することもできます。


.. 
 Namespace miscellanea
 ---------------------

ネームスペースの集まり
---------------------

.. 
 If a name is qualified (e.g. ``Base.sin``), then it can be accessed even if
 it is not exported. This is often useful when debugging.

名前が修飾されている場合（ ``Base.sin`` など）、エクスポートされていなくてもアクセスできます。
これはデバッグの際に便利です。

.. 
 Macro names are written with ``@`` in import and export statements, e.g.
 ``import Mod.@mac``. Macros in other modules can be invoked as ``Mod.@mac``
 or ``@Mod.mac``.

マクロ名は、 ``import Mod.@mac`` のように、インポートおよびエクスポートステートメントで ``@`` を使って記述されます。
他のモジュールのマクロは、 ``Mod.@mac`` または ``@Mod.mac`` として呼び出すことができます。

.. 
 The syntax ``M.x = y`` does not work to assign a global in another module;
 global assignment is always module-local.

構文 ``M.x = y`` は、別のモジュールでグローバルを割り当てることはできません。
グローバル割り当ては、常にモジュールローカルとなります。

.. 
 A variable can be "reserved" for the current module without assigning to
 it by declaring it as ``global x`` at the top level. This can be used to
 prevent name conflicts for globals initialized after load time.

変数は、上位で ``global x`` として宣言することによって、実際に割り当てることなく、
カレントモジュールの「予約済み」とすることができます。
これは、ロード後に初期化されたグローバルの名前の競合を防ぐために使用できます。

.. _man-modules-initialization-precompilation:

.. 
 Module initialization and precompilation
 ----------------------------------------

モジュールの初期化とプリコンパイル
----------------------------------------

Large modules can take several seconds to load because executing all of
the statements in a module often involves compiling a large amount of code.
Julia provides the ability to create precompiled versions of modules
to reduce this time.

To create an incremental precompiled module file, add
``__precompile__()`` at the top of your module file
(before the ``module`` starts).
This will cause it to be automatically compiled the first time it is imported.
Alternatively, you can manually call ``Base.compilecache(modulename)``.
The resulting cache files will be stored in ``Base.LOAD_CACHE_PATH[1]``.
Subsequently, the module is automatically recompiled upon ``import``
whenever any of its dependencies change;
dependencies are modules it imports, the Julia build, files it includes,
or explicit dependencies declared by ``include_dependency(path)`` in the module file(s).

For file dependencies, a change is determined by examining whether the modification time (mtime)
of each file loaded by ``include`` or added explicity by ``include_dependency`` is unchanged,
or equal to the modification time truncated to the nearest second
(to accommodate systems that can't copy mtime with sub-second accuracy).
It also takes into account whether the path to the file chosen by the search logic in ``require``
matches the path that had created the precompile file.

It also takes into account the set of dependencies already loaded into the current process
and won't recompile those modules, even if their files change or disappear,
in order to avoid creating incompatibilities between the running system and the precompile cache.
If you want to have changes to the source reflected in the running system,
you should call ``reload("Module")`` on the module you changed,
and any module that depended on it in which you want to see the change reflected.

Precompiling a module also recursively precompiles any modules that are imported therein.
If you know that it is *not* safe to precompile your module
(for the reasons described below), you should put
``__precompile__(false)`` in the module file to cause ``Base.compilecache`` to
throw an error (and thereby prevent the module from being imported by
any other precompiled module).

``__precompile__()`` should *not* be used in a module unless all of its
dependencies are also using ``__precompile__()``. Failure to do so can result
in a runtime error when loading the module.

In order to make your module work with precompilation,
however, you may need to change your module to explicitly separate any
initialization steps that must occur at *runtime* from steps that can
occur at *compile time*.  For this purpose, Julia allows you to define
an ``__init__()`` function in your module that executes any
initialization steps that must occur at runtime.
This function will not be called during compilation
(``--output-*`` or ``__precompile__()``).
You may, of course, call it manually if necessary,
but the default is to assume this function deals with computing state for
the local machine, which does not need to be -- or even should not be --
captured in the compiled image.
It will be called after the module is loaded into a process,
including if it is being loaded into an incremental compile
(``--output-incremental=yes``), but not if it is being loaded
into a full-compilation process.

In particular, if you define a ``function __init__()`` in a module,
then Julia will call ``__init__()`` immediately *after* the module is
loaded (e.g., by ``import``, ``using``, or ``require``) at runtime for
the *first* time (i.e., ``__init__`` is only called once, and only
after all statements in the module have been executed). Because it is
called after the module is fully imported, any submodules or other
imported modules have their ``__init__`` functions called *before* the
``__init__`` of the enclosing module.

Two typical uses of ``__init__`` are calling runtime initialization
functions of external C libraries and initializing global constants
that involve pointers returned by external libraries.  For example,
suppose that we are calling a C library ``libfoo`` that requires us
to call a ``foo_init()`` initialization function at runtime. Suppose
that we also want to define a global constant ``foo_data_ptr`` that
holds the return value of a ``void *foo_data()`` function defined by
``libfoo`` — this constant must be initialized at runtime (not at compile
time) because the pointer address will change from run to run.  You
could accomplish this by defining the following ``__init__`` function
in your module::

    const foo_data_ptr = Ref{Ptr{Void}}(0)
    function __init__()
        ccall((:foo_init, :libfoo), Void, ())
        foo_data_ptr[] = ccall((:foo_data, :libfoo), Ptr{Void}, ())
    end

Notice that it is perfectly possible to define a global inside
a function like ``__init__``; this is one of the advantages of using a
dynamic language. But by making it a constant at global scope,
we can ensure that the type is known to the compiler and allow it to generate
better optimized code.
Obviously, any other globals in your module that depends on ``foo_data_ptr``
would also have to be initialized in ``__init__``.

Constants involving most Julia objects that are not produced by
``ccall`` do not need to be placed in ``__init__``: their definitions
can be precompiled and loaded from the cached module image.
This includes complicated heap-allocated objects like arrays.
However, any routine that returns a raw pointer value must be called
at runtime for precompilation to work
(Ptr objects will turn into null pointers unless they are hidden inside an isbits object).
This includes the return values of the Julia functions ``cfunction`` and ``pointer``.

Dictionary and set types, or in general anything that depends on the
output of a ``hash(key)`` method, are a trickier case.  In the common
case where the keys are numbers, strings, symbols, ranges, ``Expr``,
or compositions of these types (via arrays, tuples, sets, pairs, etc.)
they are safe to precompile.  However, for a few other key types, such
as ``Function`` or ``DataType`` and generic user-defined types where
you haven't defined a ``hash`` method, the fallback ``hash`` method
depends on the memory address of the object (via its ``object_id``)
and hence may change from run to run.
If you have one of these key types, or if you aren't sure,
to be safe you can initialize this dictionary from within your
``__init__`` function.
Alternatively, you can use the ``ObjectIdDict`` dictionary type,
which is specially handled by precompilation so that it is safe to
initialize at compile-time.

When using precompilation, it is important to keep a clear sense of the
distinction between the compilation phase and the execution phase.
In this mode, it will often be much more clearly apparent that
Julia is a compiler which allows execution of arbitrary Julia code,
not a standalone interpreter that also generates compiled code.

Other known potential failure scenarios include:

1. Global counters (for example, for attempting to uniquely identify objects)
   Consider the following code snippet::

    type UniquedById
        myid::Int
        let counter = 0
            UniquedById() = new(counter += 1)
        end
    end

   while the intent of this code was to give every instance a unique id,
   the counter value is recorded at the end of compilation.
   All subsequent usages of this incrementally compiled module
   will start from that same counter value.

   Note that ``object_id`` (which works by hashing the memory pointer)
   has similar issues (see notes on Dict usage below).

   One alternative is to store both ``current_module()`` and the current ``counter`` value,
   however, it may be better to redesign the code to not depend on this global state.

2. Associative collections (such as ``Dict`` and ``Set``) need to be re-hashed in ``__init__``.
   (In the future, a mechanism may be provided to register an initializer function.)

3. Depending on compile-time side-effects persisting through load-time.
   Example include:
   modifying arrays or other variables in other Julia modules;
   maintaining handles to open files or devices;
   storing pointers to other system resources (including memory);

4. Creating accidental "copies" of global state from another module,
   by referencing it directly instead of via its lookup path.
   For example, (in global scope)::

       #mystdout = Base.STDOUT #= will not work correctly, since this will copy Base.STDOUT into this module =#
       # instead use accessor functions:
       getstdout() = Base.STDOUT #= best option =#
       # or move the assignment into the runtime:
       __init__() = global mystdout = Base.STDOUT #= also works =#

Several additional restrictions are placed on the operations that can be done while precompiling code
to help the user avoid other wrong-behavior situations:

1. Calling ``eval`` to cause a side-effect in another module.
   This will also cause a warning to be emitted when the incremental precompile flag is set.

2. ``global const`` statements from local scope after ``__init__()`` has been started (see issue #12010 for plans to add an error for this)

3. Replacing a module (or calling ``workspace()``) is a runtime error while doing an incremental precompile.

A few other points to be aware of:

1. No code reload / cache invalidation is performed after changes are made to the source files themselves,
   (including by ``Pkg.update``), and no cleanup is done after ``Pkg.rm``

2. The memory sharing behavior of a reshaped array is disregarded by precompilation (each view gets its own copy)

3. Expecting the filesystem to be unchanged between compile-time and runtime
   e.g. ``@__FILE__``/``source_path()`` to find resources at runtime,
   or the BinDeps ``@checked_lib`` macro. Sometimes this is unavoidable.
   However, when possible, it can be good practice to copy resources
   into the module at compile-time so they won't need to be found at runtime.

4. WeakRef objects and finalizers are not currently handled properly by the serializer
   (this will be fixed in an upcoming release).

5. It is usually best to avoid capturing references to instances of internal metadata objects such as
   Method, MethodInstance, MethodTable, TypeMapLevel, TypeMapEntry
   and fields of those objects, as this can confuse the serializer
   and may not lead to the outcome you desire.
   It is not necessarily an error to do this,
   but you simply need to be prepared that the system will
   try to copy some of these and to create a single unique instance of others.

It is sometimes helpful during module development to turn off incremental precompilation.
The command line flag ``--compilecache={yes|no}`` enables you to toggle module precompilation on and off.
When Julia is started with ``--compilecache=no`` the serialized modules in the compile cache are ignored when loading modules and module dependencies.
``Base.compilecache()`` can still be called manually and it will respect ``__precompile__()`` directives for the module.
The state of this command line flag is passed to ``Pkg.build()`` to disable automatic precompilation triggering when installing, updating, and explicitly building packages.
