---
title: PowerShell study notes
author: DNV825
date: 2021-01-24
category: Jekyll
layout: post
order: 1
---

文法やノウハウのメモ。

## PowerShellとは

WMF (Windows Management Framework) の一部。
Windowsシステムの運用・管理を手助けするツール群の1つ。
コマンドレット (Cmdlet) というコマンドを用い、.NET Frameworkのクラスも自由に利用できる。

コーディング規約は以下を参考。

<https://tech.guitarrapc.com/entry/2017/12/03/230119#余談--PowerShell-Core-のコーディングスタイル>

## コマンドレット (Cmdlet)

コマンドプロンプトでいう内部コマンド。
従来の内部コマンドとは異なり、テキストデータではなく .NET Frameworkのオブジェクトを出力する。

従来と同様に | でオブジェクトをパイプすることができるので、柔軟な操作が可能（プロパティを次のコマンドレットに渡したりとか。）
Unix系OSはシステム情報をテキストで保存しているが、Windowsはバイナリデータ（オブジェクト）などで保持しているので
オブジェクトを操作できるPowerShellはWindowsの操作に最適のシェルと言える。

なお、コマンドレットはデフォルトで使えるもの、モジュールをインポート、あるいはPSスナップインを追加することで利用可能になるものがある。
インポートしたモジュールに同名のコマンドレットがある場合、「モジュール名\コマンドレット名」のように呼び出すと意図したコマンドレットを実行できる。

## PowerShellのバージョン確認

スクリプトを配布する際、各自のPowerShellバージョンがバラバラでも動くようにしなければいけないため
以下の方法でバージョンを取得し、処理を分ける。

| 方法 | 意味 | 役割 |
| --- | --- | --- |
| $PSVersionTable.PSVersion | PowerShellのバージョン | 意味の通りPowerShellのバージョン。 |
| $PSVersionTable.CLRVersion | PowerShellが動いている .NET FrameworkのCLRのバージョン | PowerShellのバージョンにより、必要なCLRが異なる。CLRが異なるとできないことがあるのでそのチェックに使う。 |

例えば、Windows7に標準インストールされているPCであれば、.NET Frameworkを4.5.2以上にアップデートしなければPowerShell 5.1は使えない。

## スクリプトの実行方法

デフォルト設定でスクリプトの実行が抑制されているので、以下のコマンドを入力する。

```powershell
Set-ExecutionPolicy RemoteSigned -Scope Process
```

## PowerShellの機能

### PSドライブ

PSドライブとは、階層構造を持つ各種リソースに容易にアクセスするための手段。

コマンドプロンプトではファイルシステムのドライブしか扱えなかったが、PowerShellはレジストリ、デジタル署名、環境変数、
エイリアス、スクリプト変数、関数、WS-Management構成情報をPSドライブとして扱うことができる。

Get-DriveコマンドレットでPSドライブの一覧を取得できる。
Set-LocationコマンドレットでPSドライブを移動できる。

### PSプロバイダ

PSプロバイダとは、PSドライブを扱うためのプログラムでPSドライブの種類を表す。
たとえば、PSドライブのC:は「FileSystemプロバイダの提供するFileSystemドライブ」という形。
Get-PSProviderで利用可能なPSプロバイダを調べられる。

New-PSDriveコマンドレットでPSプロバイダを指定し、任意のPSドライブを作成することもできる。

### プロファイル

PowerShellが実行されるたびに読み込まれるスクリプトファイルのこと。
シェル変数 $profile のパスにps1ファイルを用意しておくと、起動時に読み込まれる。

### 変数 $xxx

$xxx で変数となる。
大文字小文字は区別しない。
変数名に利用でいない文字を変数名として利用する場合、${Input-Item}のように波かっこで囲む。
また、PSドライブのアイテムは変数として使える。たとえば、${C:\sample.txt}のように。
変数はVariable:ドライブからもアクセスできる。

PowerShellにおける変数のスコープは3種類。

| スコープ | 意味 |
| --- | --- |
| global | どのスコープからでも読み書き可能。 |
| private | 現在のスコープでのみ読み書き可能。 |
| script | 現在のスクリプト内でのみ読み書き可能。 |

変数名の前に $global:a のようにスコープを示すラベルをつけると変数のスコープを設定できる。
ラベルをつけない場合、それぞれのスコープに応じた変数が自動生成される。

スクリプトスコープだけちょっと理解しづらくて、以下のような動作になる。

```powershell
"&{$l = 1};$l" > l.ps1 # スクリプトの子スコープで変数$lを宣言&割り当て。
.\l.ps1                # スクリプトを呼び出しても子スコープはスコープ外なので値を出せない。

"&{$script:m = 1};$m" > m.ps1 # scriptスコープにすると
.\m.ps1                       # スクリプト内からであれば子スコープでも呼び出せる。
1
```

参照できるということは、スコープを終えても変数はなくならないんだな。

### シェル変数

PowerShellを起動する際、あるいは何らかの操作を行った際に自動的に宣言される変数。
PowerShellのコマンドが実行されるたびに値の代入が行われる「自動変数」と
ユーザーが変更することでコマンドレットなどの動作が変化する「ユーザー定義変数」がある。

基本的な自動変数は以下の通り（全部は書かない。本を見ること。）

| 自動変数 | 役割 |
| --- | --- |
| $_ | 現在のパイプラインオブジェクト。スクリプトブロックや各種ステートメント中で用いる。 |
| $Input | パイプラインに入力されたオブジェクト。 |
| $PsCmdlet | 実行中の高度な関数を表すオブジェクト。高度な関数の情報を参照したり機能を実行するのに用いる。 |
| $foreach | foreachループで現在処理中のオブジェクト。 |
| $Args | 関数、またはスクリプトに渡される未宣言パラメーターの配列。 |
| $PSBoundParameters | 関数に渡されたパラメーターとパラメーター値を格納した連想配列。 |
| $Swith | switchステートメント内の列挙子。 |
| $matches | -match演算子使用時の正規表現マッチ情報が格納された連想配列。 |
| $profile | プロファイルのパス。 |
| $HOME | ユーザーのホームディレクトリ。 |
| $PSHOME | PowerShellのインストールディレクトリ。 |
| $PSCommandPath | 現在実行中のスクリプトのパス。 |
| $PWD | カレントディレクトリ。 |
| $true | 真。 |
| $false | 偽。 |
| $null | Null。未初期化変数に格納される。 |

### 実行演算子 &

&をつけた変数を実行することができる。
同じような処理を何度も実行するけど、入力するのが面倒くさいときに使える。

```powershell
$sub = get-childitem c:\
& $sub
```

なお、以下のようにすると子スコープを作って実行することになる。

```powershell
&{get-childitem c:\}
```

### 文字列リテラル

""で囲むと文字列リテラルになる。
文字列リテラルは.NET Frameworkのstringオブジェクトになる。
また、変数が展開される「テンプレートリテラル」である。

```powershell
"$pshome is here!"
C:\Windows\System32\WindowsPowerShell\v1.0 is here!

"${pshome}is here" # スペースを入れないで変数を展開する場合は{}で囲む。
C:\Windows\System32\WindowsPowerShell\v1.0is here!

"$($env:path) is environment path." # $()で囲むと式を埋め込める。
いろいろ
```

シングルクォーテーションでも文字列リテラルを作るが、こちらは変数を展開しない。

また、@""@、@''@でヒアドキュメント（ヒア文字列）になる。

```powershell
$str = @"
Yes
We
Can!
"@

$str
Yes
We
Can!
```

### コメント #、<#～#>

PowerShellでは、#で1行コメント、<#～#>で複数行コメントを表現する。

### 特殊文字 `

エスケープ文字たち。PowerShellでは \`(アクサングラーブ) をプレフィックスにして表現する。

| 文字 | 意味 |
| --- | --- |
| \`$ | ドルマーク |
| \`n | 改行 |
| \`" | ダブルクォーテーション |
| \`0 | Null |
| \`r | キャリッジリターン |
| \`\` | アクサングラーブ |
| \`a | 警告 |
| \`t | 水平タブ |
| \`f | 用紙送り（フォームフィード） |
| \`b | バックスペース |
| \`' | シングルクォーテーション |
| \`v | 垂直タブ |

### 行継続文字 `

VB の _ みたいなもの。単独行の最後に \` をつけると次の行を同じ行にあると見なす。
スクリプトを見やすくするために使う。

```powershell
Get-ChildItem a*,c*,d*,l*,k*,o*,p* `
-Force -Recurse
```

これは次の式と等価。

```powershell
Get-ChildItem a*,c*,d*,l*,k*,o*,p* -Force -Recurse
```

### 演算子

演算子の優先順位は以下の通り。

単項演算子 > 論理否定演算子 > 算術演算子 > 比較演算子 > その他の論理演算子 > 代入演算子

PowerShellの演算子による演算で右辺と左辺の型が異なる場合、まず右辺の型が左辺の型に型変換される。

| 演算子の種類 | 演算子 | 意味 |
| --- | --- | --- |
| 算術演算子 | + | 加算 |
| | - | 減算 |
| | - | 負の数値に変換 |
| | * | 乗算 |
| | / | 除算 |
| | % | 剰余 |
| 代入演算子 | = | 右辺を左辺の変数に代入 |
| | += | 右辺の値を左辺の変数の値に加算 |
| | -= | 右辺の値を左辺の変数の値から減算 |
| | *= | 右辺の値を左辺の変数の値に乗算 |
| | /= | 右辺の値で左辺の変数の値を除算 |
| | %= | 右辺の値で左辺の変数の値を除算し、余りを左辺の変数に代入 |
| 単項演算子 | ++ | 左辺、または右辺の変数をインクリメント |
| | -- | 左辺、または右辺の変数をデクリメント |
| 比較演算子 | -eq | 左辺と右辺の値が等しい（Equalの略） |
| | -ne | 左辺と右辺の値が異なる（Not Equalの略） |
| | -gt | 左辺の値が右辺の値より大きい（Greater Thanの略） |
| | -ge | 左辺の値が右辺の値以上（Greater Than Equalの略） |
| | -lt | 左辺の値が右辺の値より小さい（Less Thanの略） |
| | -le | 左辺の値が右辺の値以下（Less Than Equalの略） |
| 論理演算子 | -and | 論理積 |
| | -or | 論理和 |
| | -xor | 排他的論理和 |
| | -not | 論理否定 |
| | ! | 論理否定（-notと同じ） |
| ビット演算子 | -band | ビット単位の論理積 |
| | -bor | ビット単位の論理和 |
| | -bxor | ビット単位の排他的論理和 |
| | -bnot | ビット単位の論理否定 |
| | -shr | 右へビットシフト（他言語の >> に相当し、2 -shr 3 のように使う） |
| | -shl | 左へビットシフト（他言語の << に相当し、2 -shl 3 のように使う） |
| 文字列演算子 | + | 文字列連結 |
| | += | 文字列追加代入 |
| | \* | 文字列の繰り返し（"a" * 6 → "aaaaaa" のように使う） |
| | -like | ワイルドカード比較（"abcde" -like "a*" のように使う） |
| | -notlike | ワイルドカード否定比較 |
| | -match | 正規表現比較 |
| | -notmatch | 正規表現否定比較 |
| | -replace | 置換演算子 |
| | -f | フォーマット演算子（右辺の値を左辺のフォーマット文字列に従って整形し、結果を文字列で返す） |
| 配列演算子 | + | 左辺の配列と右辺の配列、または単一の値を結像し、新しい配列として返す |
| | * | 左辺の配列要素を右辺の数値分だけ繰り返した配列を作って返す |
| | += | 左辺の配列の末尾に右辺の単一の値、または配列を結合する |
| | *= | 左辺の配列要素を右辺の数値分だけ繰り返した配列を作り、左辺に代入する |
| | -contains | 左辺の配列に右辺の値と等しい要素が含まれる場合に $True を返す |
| | -notcontains | 左辺の配列に右辺の値と等しい要素が含まれない場合に $True を返す |
| | -in | -contains と同様だが、左辺に値、右辺に配列を指定する |
| | -notin | -notcontains と同様だが、左辺に値、右辺に配列を指定する |
| | -split | 左辺の文字列を右辺の文字列（正規表現）またはフィルタスクリプトブロックに基づいて分割し、その結果を文字列配列として返す |
| | -split | 右辺の文字列を半角スペースで分割し、その結果を文字列配列として返す（左辺は指定しない） |
| | -join | 左辺の文字列配列を右辺の文字列を用いて結合し、その結果を文字列として返す |
| | -join | 右辺の文字列配列を結合し、その結果を文字列として返す（左辺は指定しない） |
| 型チェック演算子 | -is | 左辺が右辺の型であれば$Trueを返す（$a = 1; $a -is [int] のように用いる） |
| | -isnot | 左辺が右辺の型でなければ$Trueを返す |
| 型キャスト演算子 | -as | 左辺を右辺の型にキャストする（"2018/12/20" -as [DateTime] のように用いる） |
| アクセス演算子 | . | インスタンスメンバーへアクセスする |
| | :: | スタティックメンバーへアクセスする |

文字列の比較に -eq を用いると、大文字と小文字を区別しないで比較する。
大文字小文字を区別したいのであれば -ceq のように頭に c をつけた演算子を用いる。

-match 演算子で一致した結果はシェル変数 \$matches に格納される。
\$matches の実態は連想配列で、キーの 0 にマッチした文字列全体、 1 ～ サブ式の個数までサブマッチ文字列が格納される。

```powershell
PS> "abcde" -match "(.)(.)(.)"
True
PS> $matches

Name   Value
----   -----
3      c
2      b
1      a
0      abc
```

サブ式に名前をつけることも可能。

```powershell
PS> "abcde" -match "(?<first>.)(.)(.)"
True
PS> $matches

Name   Value
----   -----
first  a
3      c
2      b
0      abc
```

なお、$matchesには最初の一致しか格納されない。
複数の一致を取り出したい場合は regex オブジェクトを用いること。

-replace 演算子は左辺の文字列を右辺の配列の1番目の正規表現パターンに従って、2番目の変換先文字列に変換する。
サブ式が定義されている場合、それを変換先文字列で参照することも可能。
ただし、$は変数を表す記号でもあるため \`\$ にエスケープすること。

```powershell
PS> "abcde" -replace "abc", "z"
zde
PS> "abcde" -replace "(.)(.)(.)(.)(.)", "`$5`$4`$3`$2`$1"
edcba
```

-match も -replace も大文字小文字を区別しない。区別したい場合は先頭に c をつけて  -clike のように用いればよい。

-f 演算子は以下のように用いる。

```powershell
PS> 1/3
0.3333333333
PS> "{0:#0.#}" -f (1/3)
0.3
```

左辺に "{右辺の配列のインデックス:書式}" のように指定するのがポイントで、書式は .NET Framework のフォーマット文字列に準拠する。
MSDN ライブラリの「カスタム数値書式指定文字列」や「 DateTime 書式指定文字列」などのトピックを参照すると良い。

### 配列 @()

配列の定義、代入方法は以下の通り。

```powershell
PS> $arr = 1,2,3,4   # 配列の定義。
PS> $arr
1
2
3
4
PS> $arr[0]          # 配列の要素にアクセス。
1
PS> $arr = 5..9      # 連続した値を代入した配列の定義。
PS> $arr
5
6
7
8
9
PS> $arr.length      # 配列の要素数は length で取得する。
5                    # [0]～[4]で5個。
PS> $arr[2..4]       # 配列の部分集合を取り出すことも可能。これを「配列のスライス」と呼ぶ。
7
8
9
PS> $a, $b = $arr[0..1] # 配列を使った変数代入も可能。
PS> $a; $b
5
6
PS> $arr -gt 7       # 比較結果が $True の要素だけを取り出せる。
8
9
PS> $arr = @(1)      # 要素が 1 つだけの配列を作るときは@を使う。
PS> $arr = @()       # 要素が空の配列を作るときも@を使う。
```

戻り値が配列か単一のオブジェクトか不明なだが「必ず配列として変数に代入したい場合に 
@() を使うと良い。

```powershell
PS> $file = @(Get-ChildItem te*)
```

### 連想配列 @{}

スプラッティング記号 ( @ ) を用いると連想配列を作成でき、同様に変数名に @ を用いると
パラメーターとして渡すことができる。

連想配列の定義リテラルは

```powershell
$変数名 = @{キー1の名前 = キー1の値; キー2の名前 = キー2の値; ...}
```

```powershell
PS> $params = @{logname = "Application"; Newest = 5}
PS> Get-Eventlog @params # Get-Eventlog -logname "Application" -Newest 5 と同様の意味になる。
PS> $params.logname      # キー名で値を取得できる。
Application
PS> $params["Newest"]    # ブラケットでも取得可能。
5
```

連想配列は .NET Framework の Hashtable クラスのインスタンス。キーの格納順序は保証されない。

ただし、 PowerShell 3.0 から [ordered] 属性をつけて連想配列リテラルを記述すると Hashtable ではなく OrderedDictionary というキーの順序が保持される特殊な連想配列を作成できる。

また、 PowerShell 3.0 から連想配列を [pscustomobject] にキャスト可能になったため、ユーザー定義オブジェクトを簡単に作成できる。

```powershell
PS> $orderedDictionary = [ordered]@{a=1;b=2}
PS> $obj = [pscustomobject]@{Name="a b"; Belongs="cde"}
PS> $obj
Name            Belongs
----            -------
a b             cde
```

### リダイレクト >

コマンドプロンプトと同じくストリームのリダイレクトが可能で、リダイレクト演算子を用いる。リダイレクトでファイルを出力sるうと、UTF-16 で書きこまれる。文字コードを指定するのであれば Out-File コマンドレットを利用すればよい。

| リダイレクト演算子 | 効果 |
| --- | --- |
| > | ファイルにコマンドの出力を保存（内容は上書き） |
| \>\> | ファイルの末尾にコマンドの出力を追記 |
| n> | ファイルに n の出力を保存（内容は上書き） |
| n\>\> | ファイルの末尾に n の出力を追記 |
| n>&1 | n の出力をコマンドの出力に追加する。 |

| ストリーム名 | ストリーム番号 | 対応コマンドレット |
| --- | --- | --- |
| 標準 | 1（ n には指定不可） | コマンドレットすべて。明示的にするなら Write-Output を用いる |
| エラー | 2 | Write-Error |
| 警告 | 3 | Write-Warning |
| 詳細 | 4 | Write-Verbose |
| デバッグ | 5 | Write-Debug |
| 全ての出力 | * | なし |

以下、使用例。

```powershell
PS> Get-Process > proc.txt                     # proc.txtというファイルにプロセス一覧を出力。
PS> Get-Service not_existed_name 2>> error.txt # error.txtというファイルにエラーの内容を追記。
PS> Get-Service Bits,not_existed_name,DHCP 2>&1 > service.txt # エラー内容をコマンドの出力に追加し、それらをファイルに出力。
```

### パイプライン \|

PowerShell はコマンドをパイプライン記号（ | ）で連結して、連携させることができる。
つまり、コマンドの出力はパイプラインを通じて後続コマンドの入力とみなすことができる。

パイプラインにはプロパティやメソッドを持つオブジェクトが渡るので、パイプラインで渡した後続のコマンドでそれらのオブジェクトのメンバーを参照できる。

配列をパイプすると、配列の 1 つ 1 つの要素を順番に処理していき、その動作を配列の要素分繰り返す点に注意が必要。

パイプラインから入力されたオブジェクトは、コマンドレットの「パイプライン入力を許可する：true (By Value)」フラグのついたパラメーターに渡される。ほかに「パイプライン入力を許可する:true(ByPropertyName)というフラグもあり、これはパラメーターと同名のプロパティ名の値が渡される。

Where-Object コマンドレットを用いると入力オブジェクトにフィルターをかけられる。

```powershell
PS> Get-ChildItem | Where-Object { $_.Length -gt 1KB }
```

### 型

型として .NET Framework のオブジェクトを利用できる。 PowerShellでは動的型付けが行われ、内部的には正しい方として取り扱われる。

オブジェクトの型を調べるには

```powershell
PS> $a = 1
PS> $a | Get-Member
  TypeName: System.Int32
  (省略)
PS> $a.GetType().FullName
System.Int32
```

のいずれかを実行すれば良い。

ただ、動的型付けはバグの元でもあり、型を指定したいケースも多々ある。

PowerShellでは

```powershell
[型名]$変数名 = 値
```

とすることで型を定めた宣言が可能。

ちなみに、 PowerShell では `System.Int32` の「`System.`」は省略できる。簡単なエイリアスが用意されているので、そちらで型名を指定するのが良いだろう。

「ある型であるか」を調べるには -is 演算子を使えばよい。

```powershell
PS> $a = 1
PS> $a -is [int]
True
```

### 型変換

算術演算子は左辺値と右辺値の型が異なる場合、右辺値の型を左辺値の型に型変換した後に演算を行う。

明示的な型変換を行う場合、変数や値の前に `[型]` をつければよい。

```powershell
PS> [System.DateTime]"2018/12/20"
```

型キャスト出来ない場合はエラーになる。

-as 演算子を使っても型キャストできるが、型キャストできない場合こちらは `$Null` を返却する。

`[System.Void]`に型キャストすると値の出力を抑制できる。この動作は、Out-Null コマンドレットにオブジェクトをパイプするときの動作と同じ。

### インスタンスメンバーの呼び出し

PowerShell では「型アダプタ」という仕組みを用いて、オブジェクトの種類が .NET ・ COM ・ XML のノードなどであっても、それらのメンバーをドット演算子で呼び出すことができる。

オブジェクトの型・どのようなメンバーを持つかを調べるには Get-Member コマンドレットを用いる。

説明が長すぎて省略されている場合、Format-List コマンドレットにパイプすればよい。

```powershell
PS > $arr | Get-Member -Name ToString | Format-List

TypeName   : System.Int32
Name       : ToString
MemberType : Method
Definition : string ToString(), string ToString(string format), string ToString(System.IFormatProvider provider),
             g ToString(string format, System.IFormatProvider provider), string IFormattable.ToString(string forma
             stem.IFormatProvider formatProvider), string IConvertible.ToString(System.IFormatProvider provider)
```

パラメーター付きプロパティ（ C# でいうインデクサ、 VB でいうインデックス付きプロパティ）の参照方法は次の通り。

```powershell
変数など.パラメーター付きプロパティ名(パラメーター)
変数など[パラメーター]
```

XmlDocument オブジェクトの Item プロパティ（パラメーター付きプロパティ）を参照する例を示す。

```powershell
PS > $xml = [xml]"<root><x>a1</x><x>a2</x></root>"
PS > $xml.Item("root")

x
-
{a1, a2}
```

### スタティックメンバーの呼び出し

スタティックメンバー（インスタンスを作らなくてもアクセスできるメンバー）へのアクセスは :: 演算子を使う。スタティックメンバーを調べるには、Get-Member -Static をパイプすればよい。

ちなみに、PowerShell は .NET Framework の列挙体のメンバ名を string 型値と相互変換できる。

そのため、以下の 2 文は同じ意味となる。

```powershell
PS> $file.Attributes = [System.IO.FileAttributes]::Hidden
PS> $file.Attributes = "Hidden"
```

## PowerShell の文法

条件分岐や繰り返し、エラーキャッチ方法。ちなみに、式（ Expression ）と文（ Statement ）を認識すると理解しやすい。

### if ～ elseif ～ elseif

```powershell
PS> $a = 1
PS> if ($a -eq 0) {
PS>   Write-Host "zero"
PS> }
PS> elseif ($a -eq 1) {
PS>   Write-Host "one"
PS> }
PS> else {
PS>   Write-Host "???"
PS> }
```

### switch ～ { ～ ; break } default {}

```powershell
PS> $a = 6
PS> switch ($a) {
PS>   1 { Write-Host "$a is 1"; break }
PS>   2 { Write-Host "$a is 2"; break }
PS>   3 { Write-Host "$a is 3"; break }
PS>   4 { Write-Host "$a is 4"; break }
PS>   default { Write-Host "$a is other value."}
PS> }
```

break は switch を抜けるための構文。C言語とは異なり、break がなかったら次の式を評価するというわけではない。

値を指定する部分にスクリプトブロックを指定することも可能。現在評価中の値ということで、$_ を使える。

```powershell
PS> $a = 6
PS> switch ($a) {
PS>   {$_ -lt 5} { Write-Host "$_ is less than 5."; break }
PS>   {$_ -gt 5} { Write-Host "$_ is greater than 5."; break }
PS>   {$_ -eq 5} { Write-Host "$_ is 5."; break }
PS>   default { Write-Host "$_ is other value."}
PS> }
```

配列も渡すことができる。配列の要素 1 つ 1 つに対して処理を行うには、以下のようにbreak 文を書かないこと（ break で switch を抜けてしまうので 1 つ 1 つに処理できなくなってしまう。）

```powershell
PS> $a = 3..7
PS> switch ($a) {
PS>   {$_ -lt 5} { Write-Host "$_ is less than 5."}
PS>   {$_ -gt 5} { Write-Host "$_ is greater than 5."}
PS>   {$_ -eq 5} { Write-Host "$_ is 5." }
PS>   default { Write-Host "$_ is other value."}
PS> }
```

switch には以下のパラメーターを指定できる。

| パラメーター | 意味 |
| --- | --- |
| -regex | 文字列を正規表現で評価する |
| -wildcard | 文字列をワイルドカードで評価する |
| -exact | 文字列を完全一致で評価する |
| -casesensitive | 文字列の大文字小文字を区別して評価する |
| -file | 文字列をファイルから読み込んで評価する |

使用例は以下の通り。

```powershell
PS> $str = "abc"
PS> switch -wildcard($str) {
PS>   "d" { Write-Host "$strはdを含みます" }
PS>   "a*" { Write-Host "strはaから始まる文字列を含みます" }
PS> }
```

### while / do while

```powershell
$str = ""
$i = 0
while ($i -lt 3) {
  $str += "a"
  $i++
}
$str
```

```powershell
$str = ""
$i = 0
do {
  $str += "a"
  $i++
} while ($i -lt 3)
$str
```

以下は無限ループとその脱出方法。

```powershell
$str = ""
$i = 0
while ($True) {
  $str += "a"
  $i++
  if ($i -gt 2) {
    break
  }
}
$str
```

ループステートメントはネスト出来るので、内側のループを抜ける際に外側のループも一緒に抜けたい場合、あらかじめ該当箇所へラベルをつけておき、ラベルを break すればそこにジャンプできる。

ラベルは `:ラベル` のようにコロン1つから始める。

```powershell
:outer
while ($True) {
  while ($True) {
    if ($True) {
      break outer
    }
  }
}
```

### for

```powershell
for ($i = 0; $i -lt 10; $i++) {
  Write-Host $i
}
```

### foreach

配列やコレクションに対する反復処理。

```powershell
$sum = 0
foreach ($i in (0..10)) {
  $sum += $i
}
$sum
```

```powershell
foreach ($process in Get-Process) {
  $process.Name
}
```

ForEach-Object コマンドレットを利用すると以下のようになる。 foreach と % はエイリアス。パイプで渡すときはこっちを使う方が簡単。

```powershell
Get-Process | foreach { $_.Name }
Get-Process | % { $_.Name }
(Get-Process).Name
```

3 番目の例のような書き方でも行ける。これは「Get-Processの実行結果オブジェクトのプロパティを指定して取り出す」、という意味になる。

### function

PowerShell の関数は function ブロックでの実行結果、コンソールに出力されるはずの値が戻り値となる。

```powershell
function Get-Today {
  Get-Date -DisplayHint "date"
}
```

return キーワードを使うと、その値を返すとともにそこで関数を終了する。 return とだけ記述するとそこで関数が終わる。

```powershell
function Get-Today-Failed{
  return
  Get-Date -DisplayHint "date"
}
```

なお、コンソールに出力する値が複数ある場合、呼び出し元にはそれらすべての値を返す（途中で「return」したらそこまでの値を返す。「return 値」ならその値を返す。）

返したくない値が関数内で出力される場合、その値を [void] にキャストすれば返さなくなる。

```powershell
function Get-RecentDay{
  (Get-Date).Date
  (Get-Date).AddHours(-24).Date
}
PS> $ret = Get-RecentDay
PS> $ret
2018年12月21日 0:00:00
2018年12月20日 0:00:00
```

```powershell
function Get-RecentDay{
  [void](Get-Date).Date
  (Get-Date).AddHours(-24).Date
}
PS> $ret = Get-RecentDay
PS> $ret
2018年12月20日 0:00:00
```

#### 関数の引数

呼び出し側でパラメーターを関数に与えることができる。その際、自動変数 $args にパラメーター名が指定されていないものが配列として格納される。パラメーターが指定されたものはその名前で受け付ける。

```powershell
function Get-Sum{
  $sum = 0
  foreach ($arg in $args) {
    $sum += $arg
  }
  return ($sum)
}
PS> Get-Sum 1 2 3 4
10
```

パラメーターの名前と数を指定することも可能。パラメーター名を指定する場合は `-変数名` で指定する。パラメーター名を指定する場合、引数の順序は考慮不要。

```powershell
function Get-Division ($left, $right) {
  return ($left / $right)
}
PS> Get-Division 10 2
5
PS> Get-Division -right 2 -left 10
5
```

なお、渡された引数は自動変数 $PSBoundParameters に連想配列として格納されているので
これを関数内から参照しても良い。

関数のパラメーターを param キーワードに記述することもできる。この時に型を指定することもできる。

```powershell
function Get-Division{
  param([int]$left, [int]$right)
  return ($left / $right)
}
```

特殊な型として [switch] がある。これを指定すると

- 呼び出し側でパラメーターを指定していれば $True
- 呼び出し側でパラメーターを指定していなければ $False

をセットするようになる。 $False をセットしたい場合、呼び出し側はパラメーターを指定しなくてよい。

```powershell
function Check-Parameter {
  param([switch]$isCheck)
  return $isCheck
}
PS> Check-Parameter -isCheck

IsPresent
---------
True

PS> Check-Parameter

IsPresent
---------
False
```

パラメーターは通常値渡しになる。特殊な型 [ref] を指定すると参照渡しになる。

PowerShellの関数はデフォルト引数に対応している。

```powershell
function Get-Division {
  param ($left, $right = 1)
  return $left / $right
}
```

param キーワードはスクリプトファイルやスクリプトブロックにも定義できる。

#### 関数とパイプライン

パイプラインで関数に値を渡すと、自動変数 $input に入力オブジェクトが格納される。

```powershell
function Show-Value {
  $input
  Write-Host "以上"
}
PS> 7, 11, 15 | Show-Value
7
11
15
以上
```

パイプラインでオブジェクトが入力されるたびに行う処理をスクリプトブロックで定義できる。

```powershell
function Show-Value {
  begin {
    $count = 0
  }
  process {
    $count++
    Write-Host "${count}個目の値は${_}です。"
  }
  end {
    Write-Host "以上。"
  }
}
PS> 7, 11, 5 | Show-Value
1個目の
2個目の
3個目の
以上。
```

begin は最初の 1 回だけ、 end は最後の1回だけ呼び出される。process はそれ以外に呼び出される。自動変数 $_ にはその時点の入力オブジェクトが格納されている。

begin 、 process 、 end はスクリプトブロックにも定義できる。

コマンドレットパラメーターの「パイプライン入力を許可する」フラグに相当するものを記述する場合、[高度な関数](#高度な関数)の構文を使う必要がある。

#### フィルター

フィルター（filter）は関数のprocessブロックだけを取り出した簡易構文。
パイプラインからの入力を処理したい、かつbegin・endブロックが不要であればfilterを使うと便利。

```powershell
filter Get-Odd {
  if ($_ % 2 -eq 1) {
    return $_
  }
  else {
    return
  }
}
PS> 1, 2, 3, 4, 5 | Get-Odd
1
3
5
```

#### スクリプトブロック

PowerShell における匿名関数に相当するものをスクリプトブロックと呼ぶ。

```powershell
{スクリプトブロック}
```

のようなリテラルで記述できる。

スクリプトブロックは変数に代入したり、コマンドレットのパラメーターに指定したりできる。スクリプトブロックを関数のように実行したければ、呼び出し演算子 & を用いて

```powershell
&{スクリプトブロック}
```

とすればよい。

スクリプトブロックは匿名関数のようなものなので、param 、 begin 、 process 、 end ステートメントの構文は同じように使える。

#### デフォルト定義関数

`Get-ChildItem function:` のように、コマンドレットに `function:" を渡して実行すると、デフォルトで定義された関数を一覧表示できる。

```powershell
PS > Get-ChildItem function:

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        A:
Function        B:
Function        C:
Function        cd..
Function        cd\
Function        Clear-Host
Function        D:
Function        E:
Function        F:
Function        G:
Function        Get-Verb
Function        H:
Function        help
Function        I:
Function        ImportSystemModules
Function        J:
Function        K:
Function        L:
Function        M:
Function        mkdir
Function        more
Function        N:
Function        O:
Function        oss
Function        P:
Function        Pause
Function        prompt
Function        PSConsoleHostReadLine                              2.0.0      PSReadLine
Function        Q:
Function        R:
Function        S:
Function        T:
Function        TabExpansion2
Function        U:
Function        V:
Function        W:
Function        X:
Function        Y:
Function        Z:
```

### 高度な関数

関数に様々な属性を付与することで通常の関数よりも高度な機能を持たせたものを「高度な関数」と呼ぶ。本来、コマンドレットは C# で記述するのだが、高度な関数であれば PowerShell スクリプトで記述でき、かつコマンドレットと同等の機能を持たせることが可能。

通常の関数に [CmdletBinding] 属性を付与することでその関数は高度な関数となる。細かい話は [about_functions_advanced](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_functions_advanced?view=powershell-7.5) を参照。

最も基本的な構文は以下の通り。

```powershell
function 高度な関数名 {
  [CmdletBinding()]
  param (パラメーター定義)
  コマンド
}
```

ComdletBinding 属性に指定できる内容は以下の通り。

| CmdletBinding属性の属性パラメーター | 意味（カッコ内は指定できる値） |
| --- | --- |
| SupportsShouldProcess | -Whatlf 、 -Confirm パラメーターを利用可能にするかどうか（$True/$False） |
| SupportsPaging | -First 、 -Skip 、 -IncludeTotalCount パラメーターを利用可能にするかどううか（$True/$False） |
| SupportsTransactions | トランザクションが有効かどうか（$True/$False） |
| PositionalBinding | 位置パラメーターを有効にするか（$True/$False） |
| DefaultParameterSetName | パラメーターセットが解決できない場合に利用するパラメーターセット名 |
| HelpURI | Get-Help -Online で開くURL |
| ConfirmImpact | 確認プロンプトを表示する ConfirmImpact の値（None、Low、Medium、High） 自動変数 $ConfirmPreference 以上の値だと確認プロンプトが表示される |
| RemotingCapability | リモート実行時の挙動を決める RemotingCapability の値（None、OwnedByCommand、PowerShell、SupportedByCommand） |

使い方は以下の通り。

```powershell
function Get-Test {
  [CmdletBinding (SupportsShouldProcess = $True,
                  ConfirmIMpact = "Medium"]
  param (省略)
  (省略)
}
```

#### 高度な関数のパラメーター

通常の関数ではパラメーターとその属性を定義できるが、高度な関数はそれに加えてパラメーター属性というものを設定でき、パラメータごとに様々な振る舞いを指定できる。

また、パラメーター検証属性を用いることで、パラメーターとして指定された値が妥当であるかを高度な関数実行前に事前検証できる。

#### 高度な関数のヘルプ

いわゆるコメントベースのヘルプを記載できる。コメントベースのヘルプについては [about_comment_based_help](https://docs.microsoft.com/ja-jp/powershell/scripting/developer/help/examples-of-comment-based-help?view=powershell-7) を参照。

```powershell
<#
  .synopsis
  プロンプトとファイルにメッセージをエコー。

  .description
  $IsOutputLogFile が $True の場合、プロンプトとファイルにメッセージをエコーする。
  $False の場合はプロンプトだけにメッセージをエコーする。
  ファイルへのエコーは常時追記とするため、関数実行前にファイルの初期化処理が必要。

  .parameter message
  出力するメッセージ。

  .parameter foregroundColor="Gray"
  ホストに出力する文字色。

  .example
  Output-Message "Hello World"

  .notes
  何か伝えたいこととか。
#>
```

## Enum （列挙体）

<https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_enum?view=powershell-7.5>

```powershell
Enum se { A = 1; B = 2; }

[se]::A
A

[se]::A.value__
1
```

## 非同期処理

Jobs と RunSpaces

## その他ノウハウ

### オブジェクトのメンバーを一覧化

Get-Member を実行すればよい。

### インプットとアウトプット

read-host と write-host を使える。ただ、スクリプトを呼び出す場合は read-host が使えないっぽい。

### パスワードの暗号化

get-credential でも作れるが、パスワードの入力が必要となる。そのため、

```powershell
$credential = New-Object -type System.​Management.​Automation.PSCredential -argumentlit ユーザー名,パスワード
```

という形でユーザー名とパスワードを記載する。ただ、ユーザー名とパスワードは平文になるので、スクリプトの扱いには要注意。

### Windows Runtime（WinRT）を利用する

Windows Runtime (WinRT) とは、Windows 8 で実装された Modern UI スタイルのアプリケーションを実装するためのAPIのこと。PowerShell で WinRT API を利用するためには明示的な参照が必要。

以下のように参照することで、スクリプト中で WinRT API を利用することができる。

```powershell
# [class, namespace, ContentType = WindowsRuntime]
[Windows.UI.Notifications.ToastNotificationManager, Windows.UI.Notifications, ContentType = WindowsRuntime] > $null
```

`ContentType = WindowsRuntime` が重要！

以下、 1 番目の参考URLからコピペしたもの。これを改造すれば、スクリプト呼び出し時に表示内容を変更できるようになる。

```powershell
$ErrorActionPreference = "Stop"

$notificationTitle = "Notification: " + [DateTime]::Now.ToShortTimeString()

[Windows.UI.Notifications.ToastNotificationManager, Windows.UI.Notifications, ContentType = WindowsRuntime] \> $null
$template = [Windows.UI.Notifications.ToastNotificationManager]::GetTemplateContent([Windows.UI.Notifications.ToastTemplateType]::ToastText01)

# Convert to .NET type for XML manipuration
$toastXml = [xml] $template.GetXml()
$toastXml.GetElementsByTagName("text").AppendChild($toastXml.CreateTextNode($notificationTitle)) > $null

# Convert back to WinRT type
$xml = New-Object Windows.Data.Xml.Dom.XmlDocument
$xml.LoadXml($toastXml.OuterXml)

$toast = [Windows.UI.Notifications.ToastNotification]::new($xml)
$toast.Tag = "PowerShell"
$toast.Group = "PowerShell"
$toast.ExpirationTime = [DateTimeOffset]::Now.AddMinutes(5)
# $toast.SuppressPopup = $true

$notifier = [Windows.UI.Notifications.ToastNotificationManager]::CreateToastNotifier("PowerShell")
$notifier.Show($toast);
```

**参考URL**：

- <https://gist.github.com/altrive/72594b8427b2fff16431>
- <https://nyanshiba.hatenablog.com/entry/2018/03/16/014246>
- <https://docs.microsoft.com/ja-jp/windows/apps/desktop/modernize/desktop-to-uwp-enhance>
- <https://www.it-swarm-ja.tech/ja/windows-10/powershellでuwpapi名前空間を使用する/944473153>
- <https://stuncloud.wordpress.com/2020/11/24/powershell-7-1-でwinrtが死んだのでなんとかしていく/>

### PowerShell とアセンブリ

#### アセンブリとは

アセンブリとは、 .NET ランタイム環境で実行できるプリコンパイル済みコードの塊のこと。具体的には .dll ファイル、または .exe ファイルであり、 .NET プログラムは1つ以上のアセンブリで構成される。

| アセンブリの種類 | 説明 |
| --- | --- |
| プライベートアセンブリ | 1 つのアプリケーションのみの唯一のプロパティであるアセンブリ。通常はアプリケーションのルートフォルダーに保存される。 |
| パブリック/共有アセンブリ | 一度に複数のアプリケーションで使用できるdllのこと。共有アセンブリは GAC（Global Cache Assenbly）に保存される。 |
| サテライトアセンブリ | 画像などの静的オブジェクトやアプリケーションに必要なそのほかの非実行ファイルのみのこと。 |

GACは `C:\\Windows\\GAC` フォルダに格納されている。

#### PowerShell にロードされているアセンブリ

PowerShell も .NET Framework を利用しているので、あらかじめいくつかのアセンブリをロードしている。以下のコマンドで PowerShell にロードされているアセンブリを一覧化できる。

```powershell
[System.AppDomain]::CurrentDomain.GetAssemblies() | % { $_.GetName().Name }
```

#### アセンブリのロード

PowerShell 推奨の手法として Add-Type を使う。

```powershell
Add-Type -AssemblyName System.Net.Http
```

実は参照するだけでもアセンブリをロードすることが可能。しかも、 WinRT API のアセンブリは参照記法でしかロードできないっぽい。

```powershell
# [class, namespace, ContentType = WindowsRuntime]
[Windows.UI.Notifications.ToastNotificationManager, Windows.UI.Notifications, ContentType = WindowsRuntime] > $null
```

**参考URL**：

- <https://tech.guitarrapc.com/entry/2014/03/17/042253>
- <https://www.vwnet.jp/Windows/PowerShell/CheckAssemblis.htm>
