=============
 ZSHEXPN (1)
=============

名称
====

zshexpn - zsh 展開と置換

概要
====

各展開は、次の５つのステップで示された順序で実行されます:

ヒストリ展開
------------

これはインタラクティブシェルでのみ実行されます。

エイリアス展開
--------------

エイリアスは、 ``zshmisc (1)`` のエイリアスの節で説明したとおり、
コマンドラインを解釈する前に、すぐに展開されます。

プロセス置換、パラメータ展開、コマンド置換、数値展開、ブレース展開
------------------------------------------------------------------

これらの５つは、左から右の順に則って、１つのステップで実行されます。
これらの展開の後、引用符で括られていない ``\``, ``'`` および ``"`` の文字は
すべて削除されます。

ファイル名展開
--------------

``SH_FILE_EXPANSION`` オプションが設定されている場合、展開の順序は sh や ksh と
互換になります。その場合、ファイル名展開はエイリアス展開の直後、上で説明した
５つの展開の前に実行されます。

ファイル名生成
--------------

一般的にはグロビングとして知られるこの展開は、常に最後に行われます。


以下のセクションでは、各展開の詳細について説明します。

ヒストリ展開
============

ヒストリ展開では、コマンドラインで以前にタイプしたコマンドの中から単語を
使うことができます。これは綴り間違いの修正や複雑なコマンドや引数の繰り返しを
容易にします。実行の直前、それぞれのコマンドはヒストリの一覧に保存され、
リストのサイズは ``HISTSIZE`` パラメータで制御できます。最も直近のコマンド
はどのような場合でも常に保持されます。ヒストリ一覧に保存されたコマンドは、
それぞれヒストリイベントと呼ばれ、シェルが起動した時に 1 から始まる番号を
振られます。プロンプトで目にするかもしれないヒストリ番号 (``zshmisc (1)`` の
``EXAPNSION OF PROMPT SEQUENCE`` を参照) は、次のコマンドに割り当てられる
番号です。

概要
----

ヒストリ展開は histchars パラメータの最初の文字 (デフォルトは ``!``) で始め、
コマンドライン上のどこにでも書けますが、入れ子にすることはできません。
``!`` は特別な意味を抑制するためには ``\`` でエスケープするか、または
シングルクオート ('') のペアで括ることができます。ダブルクオートは使えません。
ヒストリ文字に続くのはオプションのイベント指示子 (イベント指示子の節を参照)
とオプションのワード支持子 (ワード支持子の節を参照) です; これらのどちらも
存在しない場合、ヒストリ展開は実行されません。

ヒストリ展開を含む入力行は、展開後にエコーされますが、その他の展開やコマンドが
実行される前にエコーされています。後で参照されるためのヒストリイベントとして
記録されるのは、この展開された形式です。

デフォルトでは、イベント指示子のないヒストリ参照は、そのコマンドラインの
以前の任意のヒストリ参照と同じイベントを指します; それがコマンドの唯一の
ヒストリ参照である場合は、先行するコマンドを指します。しかし
``CSH_JUNKIE_HISTORY`` が設定されている場合、すべてのイベント指示子のない
ヒストリ参照は、常に先行するコマンドを参照します。

例えば、 ``!`` は以前のコマンドのイベント指示子なので、 ``!!:1`` は常に直前の
コマンドの最初のワードを指し、 ``!!$`` は常に直前のコマンドの最後のワードを
指します。 ``CSH_JUNKIE_HISTORY`` をセットすると、 ``!!:1`` と ``!$`` は
それぞれ ``!!:1`` と ``!!$`` と同様に機能します。逆に ``CSH_JUNKIE_HISTORY``
が設定されていない場合の ``!:1`` と ``!$`` は、それぞれ現在のコマンドラインで
先行する最も近い他のヒストリ参照によって参照されているイベント、もしくは
先行する参照が存在しない場合は先行するコマンドの、最初と最後のワードを指します。

``^boo^bar`` という文字シーケンス (``^`` は実際は histchars の２番めのパラメータ
です) は、最後に実行したコマンドの ``foo`` を ``bar`` に置き換えて繰り返します。
より正確には、 ``^foo^bar^`` というシーケンスは ``!!:s^foo^bar^`` と同義であり、
すなわち ``^`` の後ろに他の修飾子 (セクション 修飾子 を参照してください) を
続けることができます。具体的な例として、例えば ``^foo^bar^:G`` はグローバルな
置換を行います。

シェルが入力の中に ``!"`` の文字シーケンスを検出した場合、ヒストリの機構は
現在のリスト (``zshmisc (1)`` を参照) が完全にパースされるまで、一時的に無効に
なります。 ``!"`` が入力から取り除かれ、後続の ``!`` は特別な意味を持たなく
なります。

少し利便性は下がりますが、より分かりやすい形式のコマンドヒストリサポートが
組み込みの fc で提供されています。

イベント指示子
--------------

イベント指示子はヒストリ一覧の中のコマンドエントリを参照します。以下のリストで
各アイテムの最初の ``!`` は、 histchars パラメータを設定することで、他の文字に
変更できることを覚えておいてください。

========= =====================================================================
!         空白、改行、 ``=`` または ``(`` が後ろに続く場合を除き、ヒストリ展開
          の開始を意味します。もしワード指示子 (``ワード指示子`` の節を参照)
          が直後に続く場合、この形式はイベント指示子のないヒストリ参照です
          (``概要`` の節を参照) 。

!!        直前のコマンドを参照します。この展開は、単独で直前のコマンドを繰り返し
          ます。

!n        コマンドラインの n 番目 を参照します。

!-n       現在のコマンドラインから n 番前を参照します。

!str      最も直近の *str* から始まるコマンドを参照します。

!?str[?]  *str* を含む最も直近のコマンドを参照します。末尾の ``?`` は、もしこの
          参照の後ろに修飾子や任意のテキストが続く場合には、 *str* の一部と
          判断されないようにするために必要です。

!#        現在のコマンドライン上での、これまでの入力を参照します。入力行は
          あたかも ``!#`` 参照と共にあったワードを含めて、補完されたかのように
          扱われます。

!{...}    隣り合う文字から、ヒストリ参照を分離させます。
========= =====================================================================

ワード指示子
------------

ワード指示子は指示された単語またはコマンドラインの単語がヒストリ参照に含まれる
べきかを示します。 ``:`` は単語指示子からイベント指定を分離します。
``^``, ``$``, ``*``, ``-`` または ``%`` で始まるワード指示子のみ、それを省略する
ことができます。ワード指示子は以下を含みます :

====== ================================================================
0      最初の入力ワード (コマンド) 。
n      n 番目の引数。
^      最初の引数。つまり 1 。
$      最後の引数。
%      ?str の検索にマッチした (最も新しい) ワード。
x-y    ワードの範囲; x のデフォルトは 0 。
\*     すべての引数、もしくは値がなれけば null 。
x\*    ``x-$`` の省略形。
x-     ``x*`` と似ているが ``$`` を省略。
====== ================================================================

Note that a `%' word designator works only when used in  one  of ``!%``,
``!:%`` or ``!?str?:%``, and only when used after a !? expansion (possibly
in an earlier command).  Anything else results in  an  error,  although
the error may not be the most obvious one.

'%' ワード指示子は ``!%``, ``!:%``, ``!?str?:%`` および (場合によっては
過去のコマンドの) ``!?`` 展開 の後の、いずれかで使われる場合にのみ機能すること
に注意してください。


Modifiers
---------
After  the  optional  word designator, you can add a sequence of one or
more of the following modifiers, each preceded by a ``:``. These  modi-
fiers  also  work  on  the  result of filename generation and parameter
expansion, except where noted.

========= ======================================================================
a         Turn a file name into an absolute path:   prepends  the  current
          directory, if necessary, and resolves any use of ``..`` and ``.`` in
          the path.  Note that the transformation takes place even if  the
          file or any intervening directories do not exist.

A         As ``a``, but also resolve use of symbolic links where possible.
          Note that resolution of ``..`` occurs before  resolution  of  sym-
          bolic  links.   This  call is equivalent to a unless your system
          has the realpath system call (modern systems do).

c         Resolve a command name into an absolute path  by  searching  the
          command path given by the PATH variable.  This does not work for
          commands containing directory parts.  Note also that  this  does
          not  usually  work as a glob qualifier unless a file of the same
          name is found in the current directory.

e         Remove all but the part of the filename extension following  the
          ``.``;  see  the  definition  of  the  filename  extension  in the
          description of the r modifier below.   Note  that  according  to
          that definition the result will be empty if the string ends with
          a ``.``.

h         Remove a trailing pathname component, leaving  the  head.   This
          works like ``dirname``.

l         Convert the words to all lowercase.

p         Print  the  new  command but do not execute it.  Only works with
          history expansion.

q         Quote the substituted  words,  escaping  further  substitutions.
          Works with history expansion and parameter expansion, though for
          parameters it is only useful if the  resulting  text  is  to  be
          re-evaluated such as by eval.

Q         Remove one level of quotes from the substituted words.

r         Remove a filename extension leaving the root name.  Strings with
          no filename extension are not altered.  A filename extension  is
          a ``.`` followed by any number of characters (including zero) that
          are neither ``.`` nor ``/`` and that continue  to  the  end  of  the
          string.  For example, the extension of ``foo.orig.c`` is ``.c``, and
          ``dir.c/foo`` has no extension.

s/l/r[/]  Substitute r for l as described below.  The substitution is done
          only  for  the  first string that matches l.  For arrays and for
          filename generation, this applies to each word of  the  expanded
          text.  See below for further notes on substitutions.

          The  forms ``gs/l/r`` and ``s/l/r/:G`` perform global substitution,
          i.e. substitute every occurrence of r for l.  Note that the g or
          :G must appear in exactly the position shown.

          See further notes on this form of substitution below.

&         Repeat  the  previous  s  substitution.  Like s, may be preceded
          immediately by a g.  In parameter expansion the  &  must  appear
          inside braces, and in filename generation it must be quoted with
          a backslash.

t         Remove all leading pathname components, leaving the tail.   This
          works like ``basename``.

u         Convert the words to all uppercase.

x         Like  q, but break into words at whitespace.  Does not work with
          parameter expansion.
========= ======================================================================

The s/l/r/ substitution works as follows.   By  default  the  left-hand
side  of  substitutions  are  not patterns, but character strings.  Any
character can be used as the delimiter in place of ``/``.   A  backslash
quotes   the   delimiter   character.    The   character ``&``,  in  the
right-hand-side r, is replaced by the text from the  left-hand-side  l.
The ``&`` can  be  quoted with a backslash.  A null l uses the previous
string either from the previous l or from the contextual scan string  s
from ``!?s``.  You can omit the rightmost delimiter if a newline immedi-
ately follows r; the rightmost ``?`` in a context scan can  similarly  be
omitted.  Note the same record of the last l and r is maintained across
all forms of expansion.

Note that if a ``&`` is used within glob qualifers an extra backslash  is
needed as a & is a special character in this case.

If  the  option HIST_SUBST_PATTERN is set, l is treated as a pattern of
the usual form described in  the  section  FILENAME  GENERATION  below.
This can be used in all the places where modifiers are available; note,
however, that in globbing qualifiers parameter substitution has already
taken  place,  so parameters in the replacement string should be quoted
to ensure they are replaced at the correct time.  Note also  that  com-
plicated  patterns  used  in  globbing qualifiers may need the extended
glob qualifier notation (#q:s/.../.../) in order for the shell to  rec-
ognize the expression as a glob qualifier.  Further, note that bad pat-
terns in the substitution are not subject to the NO_BAD_PATTERN  option
so will cause an error.

When  HIST_SUBST_PATTERN  is set, l may start with a # to indicate that
the pattern must match at the start of the string  to  be  substituted,
and a % may appear at the start or after an # to indicate that the pat-
tern must match at the end of the string to be substituted.  The % or #
may be quoted with two backslashes.

For  example,  the following piece of filename generation code with the
EXTENDED_GLOB option::

       print *.c(#q:s/#%(#b)s(*).c/'S${match[1]}.C'/)

takes the expansion of \*.c and  applies  the  glob  qualifiers  in  the
(#q...)  expression, which consists of a substitution modifier anchored
to the start and end of each word (#%).  This turns  on  backreferences
((#b)),  so  that  the  parenthesised subexpression is available in the
replacement string as ${match[1]}.  The replacement string is quoted so
that the parameter is not substituted before the start of filename gen-
eration.

The following f, F, w and W modifiers work only with  parameter  expan-
sion and filename generation.  They are listed here to provide a single
point of reference for all modifiers.

======== =======================================================================
f        Repeats the immediately (without  a  colon)  following  modifier
         until the resulting word doesn't change any more.

F:expr:  Like  f,  but repeats only n times if the expression expr evalu-
         ates to n.  Any character can be used instead  of  the ``:``;  if
         ``(``,  ``[``,  or ``{`` is used as the opening delimiter, the closing
         delimiter should be ``)``, ``]``, or ``}``, respectively.

w        Makes the immediately following modifier work on  each  word  in
         the string.

W:sep:   Like  w  but  words are considered to be the parts of the string
         that are separated by sep. Any character can be used instead  of
         the ``:``; opening parentheses are handled specially, see above.
======== =======================================================================

PROCESS SUBSTITUTION
====================



PARAMETER EXPANSION
===================


'$' はパラメータ展開をのために使われます。配列、連想配列および配列の個々の要素
にアクセスするための添字記法を含むパラメータの詳細については zsh-param (1) を
参照してください。

``SH_WORD_SPLIT`` オプションが設定されていない限り、引用符で囲まれていない
パラメータの単語は自動的に空白文字で分割されないという事に注意してください;
詳細については以下にある、このオプションのリファレンスを参照してください。
これは、他のシェルとの重要な違いです。

以下に記載されているパターンを必要とする展開では、パターンの形式はファイル名
生成で使われるものと同じです; 'ファイル名生成' の節を参照してください。
これらのパターンは、任意の置換の置換テキストと同様に、それ自信がパラメータ展開や
コマンド置換、算術展開の対象であることに注意してください。
以下の操作に加えて、`履歴展開' の節の `修飾子' の節で説明されているコロン修飾子を
適用することができます: 例えば、 ``${i:s/foo/bar/}`` は展開されたパラメータ
``$i`` に対して文字列置換を行います。

${name}
-------

パラメータ *name* の値がもしあれば、置き換えられます。展開に *name* の一部として
解釈されるべきではない文字、数字またはアンダースコアが続くようにする場合は、
括弧が必要です。
また、より複雑な置き換えの形式の場合、通常は括弧が必要です; 例外として、
単一の添字、名前の後ろにコロン修飾子が現れる場合、または名前の前に ``'^'``,
``'='``, ``'~'``, ``'#'`` および ``'+'`` のいずれかが現れる場合、これらは
いずれも括弧があっても無くても動作しますが、 ``KSH_ARRAYS`` オプションが
設定されていない場合のみ適用されます。

もし *name* が配列パラメータで、 ``KSH_ARRAYS`` オプションが設定されていない
場合、 *name* の各要素の値は、単語ごとに一つの要素として置き換えられます。
そうでなければ一つの単語だけ置き換えられます; ``KSH_ARRAYS`` が有効の場合、
これは配列の最初の要素です。
``SH_WORD_SPLIT`` オプションが設定されていない限り、結果に対してフィールド
分割は行われません。 ``=`` フラグと ``s:string:`` も参照してください。

${+name}
--------

*name* が値の設定されたパラメータの名前ならば、 ``'1'`` で置き換えられ、
そうでない場合は ``'0'`` で置き換えられます。

Example ::

    % a=foo
    % echo ${+a}
    1
    % echo ${+b}
    0

${name-word}, ${name:-word}
----------------------------

*name* に値が設定されている、もしくは２つ目の形式で Non Null の場合、その値で
置き換えられます。そうでない場合は *word* で置き換えられます。
２つ目の形式では *name* を省略することができ、その場合は常に *word* で
置き換えられます。

Example ::

    % a=foo
    % echo ${a-hoge}
    foo
    % echo ${b-hoge}
    hoge
    % b=""
    % echo ${b-hoge}
        <-- 空文字列
    % echo ${b:-hoge}
    hoge
    %

${name+word}, ${name:+word}
----------------------------

*name* に値が設定されている、もしくは２つ目の形式で Non Null の場合、 *word* で
置き換えられます。そうでない場合は空文字で置き換えられます。

Example ::

    % a=foo
    % echo ${a+hoge}
    hoge
    % echo ${b+hoge}
        <-- 空文字列
    % b=""
    % echo ${b+hoge}
    hoge
    % echo ${b:+hoge}
        <-- 空文字列
    %

${name=word}, ${name:=word}, ${name::=word}
-------------------------------------------

１つ目の形式では *name* に値が設定されていない場合に *word* を設定します。
２つ目の形式では *name* に値が設定されていない、または Null の場合に *word* を
設定します。そして、３つ目の形式では *name* を無条件に *word* を設定します。
すべての形式でパラメータの値で代替されます。

Example ::

    % echo $a
        <-- 空文字列
    % echo ${a=foo}
    foo
    % b=""
    % echo ${b:=foo}
    foo
    % echo ${b:=hoge}
    foo
    % echo ${b::=hoge}
    hoge

${name?word}, ${name:?word}
---------------------------

１つ目の形式で *name* に値が設定されている場合、もしくは２つ目の形式で *name* に
値が設定されていて且つ Non Null の場合、その値に置き換えられます; そうでない場合
*word* を出力してシェルを終了します。対話式シェルの場合は代わりにプロンプトに
戻ります。 *word* が省略された場合、標準のメッセージが出力されます。

Example ::

    % a=foo
    % echo ${a?hoge}
    foo
    % echo ${b?hoge}
    zsh: b: hoge
    % b=""
    % echo ${b?hoge}
        <-- 空文字列
    % echo ${b:?hoge}
    zsh: b: hoge

上の、変数をテストして別の *word* に置き換える式のいずれでも、 *word* の値に
標準のシェルのクォートを使用して、 ``SH_WORD_SPLIT`` オプションと ``=`` フラグに
よって選択的に分割を上書きできますが、 ``s:string:`` フラグでは分割しません。

次の式では、 *name* が配列であり且つ置換文字列がクォートされていない場合、
もしくは ``(@)`` フラグまたは ``name[@]`` の記法が使われている場合、配列の
各要素ごとにマッチングと置換が実行されます。

${name#pattern}, ${name##pattern}
---------------------------------

*pattern* が *name* の値の先頭にマッチする場合、 *name* の値のマッチした部分が
削除された値に置き換えられます; そうでない場合は *name* の値そのものに置き換え
られるだけです。
１つ目の形式では最も短い一致が選ばれ、２つ目の形式では最も長い一致が選ばれます。

Example ::

    % str=abrakadabra
    % echo ${str#a*b}
    rakadabra
    % echo ${str##a*b}
    ra

${name%pattern}, ${name%%pattern}
---------------------------------

*pattern* が *name* の値の末尾にマッチする場合、 *name* の値のマッチした部分が
削除された値に置き換えられます; そうでない場合は *name* の値そのものに置き換え
られるだけです。
１つ目の形式では最も短い一致が選ばれ、２つ目の形式では最も長い一致が選ばれます。

Example ::

    % str=abrakadabra
    % echo ${str%r*a}
    abrakadab
    % echo ${str##r*a}
    ab

${name:#pattern}
----------------

*pattern* が *name* の値にマッチする場合、空文字列に置き換えられます; そうでない
場合は *name* の値そのものに置き換えられるだけです。
*name* が配列の場合、マッチした要素は削除されます (マッチしない要素をを削除する
ためには ``(M)`` フラグを使います) 。

Example 1 ::

    % str=abrakadabra
    % echo ${str:#a*a}
        <-- 空文字列
    % echo ${str:#a*z}
    abrakadabra

Example 2 ::

    % ary=(foo bar buz)
    % echo ${ary:#foo}
    bar buz
    % echo ${(M)ary:#foo}
    foo

${name:offset}, ${name:offset:length}
-------------------------------------

この構文は ``$name{start,end}`` の形式でパラメータに添字を指定するのと同様の
効果がありますが、他のシェルと互換性があります。 *offset* と *length* は
どちらも添字のコンポーネントとは異なる解釈をされることに注意してください。

*offset* が負の値でなく、そして変数 *name* の値がスカラーである場合には、
文字列の最初の文字から *offset* 文字目の位置から始まる内容に置き換えられ、
また *name* が配列ならば最初の要素から *offset* 個目の要素から始まる要素の
配列に置き換えられます。
*length* が指定された場合はその数の分だけの文字や要素に置き換えられ、そうでない
場合はスカラーや配列の残りの要素すべてになります。

正の *offset* は常に最初の文字または配列の最初の要素からのオフセット文字数または
要素数として扱われます (これは zsh ネイティブの添字の表記と異なります) 。
したがって ``0`` は ``KSH_ARRAYS`` オプションの設定に関わらず、最初の文字または
要素を指します。

負のオフセットはスカラーまたは配列の最後から逆方向に数えるので、 ``-1`` は
最後の文字または要素に対応…という感じです。

*length* は常にそのまま長さとして扱われるため、負の値を設定することはできません。
``MULTIBYTE`` オプションはこれに従い、すなわちマルチバイト文字をオフセットや
長さを適切にカウントします。

*offset* と *length* はスカラ代入と同様にシェル置換での設定を受け付け、さらに
その後は算術評価の対象になります。したがって、例えば ::

    print ${foo:3}
    print ${foo:1 + 2}
    print ${foo:$(( 1 + 2 ))}
    print ${foo:$(echo 1 + 2)}

これらはすべてが同じ効果、つまり ``$foo`` の置換がスカラー以外を返す場合、
４文字目から始まる文字列を取り出し、置換が配列を返す場合は４番目の要素から始まる
配列を返します。オプション ``KSH_ARRAYS`` を使う場合、 ``$foo`` は常に
(オフセット構文の使用とは関係なく) スカラーを返し、 ``$foo[*]:3`` という形式は
foo という名前の配列の要素を取り出す必要があることに注意してください。

*offset* が負の値の場合、 ``-`` は ``:`` の直後に現れると ``${name:-word}`` の
置換の形式を表すため、使用できません。その代わり ``-`` の前に空白を挿入できます。
また、 *offset* と *length* のいずれも英字や ``&`` で始めると、それらは履歴
スタイルの修飾子を表すので、使用できません。
変数から値を代入する場合、推奨するアプローチは、意図を明らかにするために ``$`` を
付けて行うことです (パラメータ置換は簡単には読み取りづらくなります); しかし
算術置換が行われるように、式 ``${var:offs}`` は、 *offs* からオフセットを取得して
置換を行います。

他のシェルとのさらなる互換性のために、配列のためのオフセット 0 の特殊なケースが
あります。これは通常、配列の一番初めの要素にアクセスします。しかし、置換が
``$@`` や ``$*`` のような位置パラメータ配列を指している場合、オフセット 0 は
代わりに ``$0`` を指し、オフセット 1 は ``$1`` を指し、のようになります。
言い換えると、位置パラメータ配列は事実上、 ``$0`` を先頭に追加して拡張されます。
したがって、 ``${*:0:1}`` は ``$0`` に、 ``${*:1:1}`` は ``$1`` に
置き換えられます。

${name/pattern/repl}, ${name//pattern/repl}
-------------------------------------------

*name* パラメータの展開後の値の中で、 *pattern* に可能な限り長い一致を、文字列
*repl* で置き換えます。１つ目の形式では最初に出現した一致のみを、２つ目の
形式ではすべての一致を置き換えます。
*pattern* と *repl* はどちらも ``${name/$opat/$npat}`` のような式が動作するように
二重引用符で括られた置換の対象になりますが、 ``GLOB_SUBST`` オプションを
設定されているか、 ``$opat`` を代わりに ``${~opat}`` のように置換されていない
限り、 ``$opat`` の中のパターン文字は特別扱いされないという通常のルールに
注意してください。 ::

    $ foo="twinkle twinkle little star"
    $ sub="t*e"

*pattern* は ``#`` で始めることができ、その場合は文字列の先頭に一致する必要が
あります。 ``%`` で始めることもでき、その場合は文字列の末尾に一致する必要が
あります。 ``#%`` で始めることもでき、この場合は文字列全体に一致する必要が
あります。 *repl* は空文字列でも良く、その場合は最後の ``/`` も省略できます。
引用符で括る場合は、最後の ``/`` の前に一つバックスラッシュが必要です; ``/`` が
置換されたパラメータの中に現れる場合、これは必要ありません。 また ``#`` 、 ``%``
および ``#%`` は、置換されたパラメータ内に現れる場合は、たとえ先頭であっても
アクティブではない事にも注意してください。 ::

    $ foo="twinkle twinkle little start"
    $ sub="#*le"
    $ rep="spy"
    $ print ${foo//${~sub}/$rep}
    zsh: bad pattern: #*le
    $ sub="*le"
    $ print ${foo//#${~sub}/$rep}
    spy star

最初の ``/`` の前には ``:`` を付けることができ、その場合は、ワード全体が一致する
場合だけ一致が成功します。下記の ``I`` と ``S`` パラメータ展開フラグの影響にも
注意してください。 ``M`` 、 ``R`` 、 ``B`` 、 ``E`` および ``N`` フラグは有用では
ありません。

例えば、 ::

    foo="twinkle twinkle little star" sub="t*e" rep="spy"
    print ${foo//${~sub}/$rep}
    print ${(S)foo//${~sub}/$rep}

この場合、 ``~`` は **$sub** のテキストがプレーンテキストではなく、パターン文字列
として扱われることを保証します。１つ目のケースでは **t\*e** の最も長い一致が置換
され、結果は **'spy star'** に、一方、二つ目のケースでは最短一致が取られ、結果は
**'spy spy lispy star'** になります。


${#spec}
--------

*spec* が前述の置換のいずれかである場合、結果の文字列の代わりにその文字列の長さ
に置き換えられます。 *spec* が配列の場合、配列の要素数に置き換えられます。
以下に示す ``'^'`` 、 ``'='`` および ``'~'`` の形式と組み合わせる場合、これらは
``'#'`` の左側になければいけないことに注意してください。

${^spec}
--------

*spec* を評価するためには、 ``RC_EXPAND_PARAM`` をオンにします; ``^`` を２つに
した場合、オフになります。
このオプションが設定されている場合、パラメータ ``xx`` に ``(a b c)`` が設定されて
いる ``foo${xx}bar`` 形式の配列の展開結果は、 **'fooa b cbar'** の代わりに、
**'fooabar foobbar foocbar'** に置き換えられます。
したがって、空の配列の場合はすべての引数が削除されることに注意してください。

内部的には、このような展開ではそれぞれ、ブレース展開と同等のリストに変換されます。
例えば、 ``${^var}`` は ``{$var[1],$var[2],...}`` のようになり、下記の'ブレース
展開'のセクションで記述されている通りに処理されます。単語分割が有効になっている
場合、 ``$var{N}`` それ自体が個別の配列に分割されることがあります。

${=spec}
--------

*spec* の評価の際に、パラメータが二重引用符に括られているかどうかに関わらず、
``SH_WORD_SPLIT`` のルールを使って単語分割を行います; ``=`` を２つにした場合、
オフになります。
これは ``IFS`` を区切り文字として、パラメータを置換前に個別の単語に分割すること
を強制します。これは他のほとんどのシェルではデフォルトで行われます。

*name* への代入が行われる前に、 *spec* の代入形式の中の *word* に分割が適用されて
いることに注意してください。これは ``A`` フラグ付き配列代入の結果に影響します。

${~spec}
--------

*spec* を評価するために ``GLOB_SUBST`` オプションをオンにします; ``~`` を２つに
した場合は。オフになります。このオプションが設定されている場合、展開された結果の
文字列は、条件文内の ``=`` や ``!=`` 演算子の右側のように、ファイル名の展開や
ファイル名生成およびパターンマッチの文脈であればどこでも、パターンとして解釈されます。

入れ子になった置換の場合、 ``~`` の効果はカレントレベルの置換に対して適用される
ことに注意してください。

If a **${...}** type ...

Note that double ...


パラメータ展開フラグ
--------------------



ルール
------


例
----


COMMAND SUBSTITUTION
====================

ARITHMETIC EXPANSION
====================

BRACE EXPANSION
===============

FILENAME EXPANSION
==================

Dynamic named directories
-------------------------

Static named directories
------------------------

'=' expansion
-------------

Notes
-----

FILENAME GENERATION
===================

Glob Operators
--------------

ksh-like Glob Operators
-----------------------

Precedence
----------

Globbing Flags
--------------

Approximate Matching
--------------------

Recursive Globbing
------------------

Glob Qualifiers
---------------


.. END
