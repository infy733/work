.. _man-documentation:

.. 
 ***************
  Documentation
 ***************

******************
 ドキュメンテーション
******************

.. 
 Julia enables package developers and users to document functions, types and
 other objects easily via a built-in documentation system since Julia 0.4.

Juliaは、パッケージ開発者やユーザーがJulia 0.4以降のビルトインドキュメントシステムを使用して、
関数、型およびその他のオブジェクトを簡単にドキュメント化することができます。

.. 
  .. tip::

    This documentation system can also be used in Julia 0.3 via the
    `Docile.jl <https://github.com/MichaelHatherly/Docile.jl>`_ package; see
    the documentation for that package for more details.


.. tip::

このドキュメントシステムは、Julia 0.3で `Docile.jl <https://github.com/MichaelHatherly/Docile.jl>`_ 
パッケージで使用できます。詳細については、パッケージの説明を参照してください。

.. 
 The basic syntax is very simple: any string appearing at the top-level right
 before an object (function, macro, type or instance) will be interpreted as
 documenting it (these are called *docstrings*). Here is a very simple example:

基本的な構文はとても単純です。オブジェクト（関数、マクロ、型、またはインスタンス）の直前に最上位に記述される文字列は、
それらをドキュメント化すると解釈されます（これらはdocstringと呼ばれています）。以下は簡単な例です。:

.. doctest::

    "Tell whether there are too foo items in the array."
    foo(xs::Array) = ...

.. 
 Documentation is interpreted as `Markdown <https://en.wikipedia.org/wiki/Markdown>`_,
 so you can use indentation and code fences to delimit code examples from text.
 Technically, any object can be associated with any other as metadata;
 Markdown happens to be the default, but one can construct other string
 macros and pass them to the ``@doc`` macro just as well.

ドキュメントは `Markdown <https://en.wikipedia.org/wiki/Markdown>`_ として解釈されるため、
インデントとコードフェンスを使用してテキストのコード例を区切ることができます。
技術的には、どのオブジェクトも他のメタデータと関連付けることができます。
マークダウンはデフォルトとなりますが、他の文字列マクロを作成して ``@doc`` マクロに渡すこともできます。

.. 
 Here is a more complex example, still using Markdown:

以下はMarkdownを使用した複雑な例です。:

.. code-block:: julia

    """
        bar(x[, y])

    Compute the Bar index between `x` and `y`. If `y` is missing, compute
    the Bar index between all pairs of columns of `x`.

    # Examples
    ```julia
    julia> bar([1, 2], [1, 2])
    1
    ```
    """
    function bar(x, y) ...

.. 
 As in the example above, we recommend following some simple conventions when writing
 documentation:

上記の例のように、ドキュメントを書くときのいくつかの簡単な慣習に沿うことをお勧めします。:

.. 
 1. Always show the signature of a function at the top of the documentation,
    with a four-space indent so that it is printed as Julia code.

    This can be identical to the signature present in the Julia code
    (like ``mean(x::AbstractArray)``), or a simplified form.
    Optional arguments should be represented with their default values (i.e. ``f(x, y=1)``)
    when possible, following the actual Julia syntax. Optional arguments which
    do not have a default value should be put in brackets (i.e. ``f(x[, y])`` and
    ``f(x[, y[, z]])``). An alternative solution is to use several lines: one without
    optional arguments, the other(s) with them. This solution can also be used to document
    several related methods of a given function. When a function accepts many keyword
    arguments, only include a ``<keyword arguments>`` placeholder in the signature (i.e.
    ``f(x; <keyword arguments>)``), and give the complete list under an ``# Arguments``
    section (see point 4 below).

1. 文書の上部には、関数のシグネチャを常に表示し、4桁のインデントをつけてJuliaコードとして表示します。
   
   これは、Juliaコード（ ``mean(x::AbstractArray)`` など）、または簡略化されたフォームのシグネチャと同一であると言えます。
   可能な場合は、実際のJulia構文に従い、オプションの引数をデフォルト値（例えば ``f(x, y=1)`` ）で表す必要があります。
   デフォルト値を持たないオプションの引数は角括弧（ ``f(x[, y])`` や ``f(x[, y[, z]])`` など）で囲んでください。
   別の解決方法は、複数の行を使用することで、オプション引数を持たない行と、それ以外の行を記述する方法です。
   この方法は、関数の関連する複数のメソッドをドキュメント化する際にも使用できます。関数が多くのキーワード引数を許容する場合、
   シグネチャ内に ``<keyword arguments>`` プレースホルダ（ ``f(x; <keyword arguments>)`` など）を記述し、
   ``# Arguments`` セクション配下に完全なリストを与える必要があります（4点目を参照）。

.. 
 2. Include a single one-line sentence describing what the function does or what the
    object represents after the simplified signature block. If needed, provide more details
    in a second paragraph, after a blank line.

    The one-line sentence should use the imperative form ("Do this", "Return that") instead
    of the third person (do not write "Returns the length...") when documenting functions.
    It should end with a period. If the meaning of a function cannot be summarized easily,
    splitting it into separate composable parts could be beneficial (this should not be
    taken as an absolute requirement for every single case though).

2. 簡略化されたシグネチャブロックの後に、関数が何を表すか、またはオブジェクトが表すものを記述する1行の文を含めてください。
   必要に応じて、空白行の後の2段落目に詳細を入力してください。
   
   関数をドキュメント化する場合は、1行文は、三人称（Returns the length…のような文）ではなく、
   命令形（Do thisやReturn thatなどの形）で記述する必要があります。そして文はピリオドで終わる必要があります。
   関数の意味を簡単に要約することができない場合、別々の構成可能なパーツに分割することができます
   （ただし、個別のケースごとに絶対的な要件とはみなされません）。

.. 
 3. Do not repeat yourself.

    Since the function name is given by the signature, there is no need to
    start the documentation with "The function ``bar``...": go straight to the point.
    Similarly, if the signature specifies the types of the arguments, mentioning them
    in the description is redundant.

3. 繰り返しをしないようにしてください。
  
   関数名はシグネチャによって与えられるため、 "The function ``bar``..." のようにドキュメントを開始する必要はありません。
   必要なことだけを記述してください。同様に、シグネチャが引数の型を指定している場合、それらを概要に記述することは冗長的です。 

.. 
 4. Only provide an argument list when really necessary.

    For simple functions, it is often clearer to mention the role of the arguments directly
    in the description of the function's purpose. An argument list would only repeat
    information already provided elsewhere. However, providing an argument list can be a good
    idea for complex functions with many arguments (in particular keyword arguments).
    In that case, insert it after the general description of the function, under
    an ``# Arguments`` header, with one ``*`` bullet for each argument. The list should
    mention the types and default values (if any) of the arguments::

4. 本当に必要なときにのみ引数リストを与えてください。
    
   単純な関数の場合、関数の目的の記述の中で、引数の役割について直接言及することはしばしば便利です。
   引数リストは既に他の場所で提供されている情報を繰り返すだけです。しかし、引数リストを提供することは、
   多くの引数（特にキーワード引数）を持つ複雑な関数に対しては有効です。その場合は、関数の一般的な説明の後に
   ``# Arguments`` ヘッダの下に、各引数に1つの「*」付きで挿入します。リストには、引数の型とデフォルト値（存在する場合）が
   記載されている必要があります。::
   
       """
       ...
       # Arguments
       * `n::Integer`: the number of elements to compute.
       * `dim::Integer=1`: the dimensions along which to perform the computation.
       ...
       """

.. 
 5. Include any code examples in an ``# Examples`` section.

    Examples should, whenever possible, be written as *doctests*. A *doctest* is a fenced
    code block (see `Code blocks`_) starting with `````jldoctest`` and contains any number of
    ``julia>`` prompts together with inputs and expected outputs that mimic the Julia REPL.

    For example in the following docstring a variable ``a`` is defined and the expected
    result, as printed in a Julia REPL, appears afterwards::

5. ``# Examples`` セクションにコードの例を含めてください。
