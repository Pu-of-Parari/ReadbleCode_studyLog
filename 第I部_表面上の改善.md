# 第I部 / 表面上の改善

まず手始めにすることのできる改善方法。

適切な命名、優れたコメント、キレイなフォーマット...これらはプログラムの変更やリファクタリングしなくても「入れ替え」が可能。しかし、**コードのすべての行に影響する**大切なテーマである。

## 名前に情報を詰め込む

名前＝短いコメント

短くても良い名前を付ければ、それだけ多くの情報を伝えることができる。

**カギとなる考え：**

> 名前に情報を詰め込む

### 明確な単語を選ぶ

「名前に情報を詰め込む」には明確な単語を選ばなければならない。「空虚」はNG。

例えば、「get」はあまり明確な単語ではない。

- メソッドの悪い例：

  ```python
  def GetPage(url):
  	・・・
  ```

  「get」という単語が曖昧。ページをどこから取ってくる？ローカルキャッシュから？データベースから？インターネットから？インターネットからなら `DownloadPage()`や `FetchPage()`の方が明確。

- BinaryTreeクラスの悪い例：

  ```c++
  class BinaryTree {
  	int Size();
  	・・・
  };
  ```

  Size()は何を返す？ツリーの高さ？ノード数？ツリーのメモリ消費量？目的に適した明確な名前を付けるなら、それぞれ `Height()`・`NumNodes()`・`MemoryBytes()`のようにするのが良い。

- Threadクラスの悪い例：

  ```c++
  class Thread {
  	void Stop();
  	・・・
  }
  ```

  Stopでも良いが、動作に合わせてもっと明確な名前を付けた方がさらに良い。取り消しができない思い操作なら`Kill()`、あとから`Resume()`できるなら、`Pause()`にするなど。

#### もっと「カラフル」な単語を探す

シソーラス(類語辞典)を活用したり、友人にもっといい名前がないかきいて適切な単語を模索してみよう

- カラフルなバージョンの単語例
  - send -> deliver, dispatch, announce, distribute, route
  - find -> search, extract, locate, recover
  - start -> launch, create, begin, open
  - make -> create, set up, build, generate, compose, add, new

ただし、やりすぎには注意。時に違いを考えなくてはいけなくなってしまう。例として、PHPの `explode()`関数は `split()`と何が違う？名前だけでは違いがわからない

**キーとなる考え：**

> 気取った言い回しよりも明確で正確な方がいい。

### tmpやretvalなど汎用的な名前を避ける

tmp・retval・fooのような名前を付けるのは、「名前のことなんて考えていませんよ」と言っているようなもの。このような空虚な名前ではなく、**エンティティの値や目的を表した名前**を選ぼう。

いい名前 = 変数の目的や値を表すもの。例えば、`vの2乗の合計`の変数名は `sum_squares`にする。こうすれば変数の目的も事前に伝わるし、バグの発見にも役立つ。

ただし、汎用的な名前に意味がないわけではない。

#### tmpについて

- 汎用的な名前をうまく使った例：

  2つの変数をスワップする古典的な操作を考える

  ```c++
  if (right < left) {
  	tmp = right;
  	right = left;
  	left = tmp;
  }
  ```

  このような場合、tmpという名前で全く問題ない。この変数の目的は、情報の一時的な保管。さらに生存期間はわずか数行。tmpという名前で「この変数にはほかに役割はない」という明確な意味を伝えている。

  つまり、他の関数に渡されたり、何度も書き換えられたりはしない、ということ。

- 汎用的な名前をうまく使えていない例：

  ```c++
  String tmp = user.name();
  tmp += " " + user.phone_number();
  tmp += " " + user.email();
  ・・・
  template.set("user_info", tmp);
  ```

  生存期間は短いが、この変数で一番大切なことは「一時的な保管」ではないため`tmp`と命名するのはナンセンス。わかりやすくするためには、`user_info`のような名前が良いだろう。

- `tmp_file`や`tmp_data`のように `_`の後ろに種類を記述するのも一つの良いアイデア。

**アドバイス：**

> tmpという名前は、生存期間が短くて、一時的な保管が最も大切な変数にだけ使おう。

#### ループイテレータについて

`i`・`j`・`k`・`iter`などの名前は、汎用的な名前だがこれだけで「ぼくはイテレータです」という意味になるため、インデックスやループイテレータで使用して問題ない。

- (Tips) ループイテレータのインデックスが複数ある時の命名の例

  クラブに所属しているユーザを調べるループを考える

  ```python
  for (int i = 0; i < clubs.size(); i++):
  	for (int j = 0; j < clubs[i].members.size(); j++):
  		for (int k = 0; k < users.size(); k++):
  			if (clubs[i].members[k] == users[j]):
  				・・・
  ```

  if文にある`members[]`と`users[]`のインデックスが逆になっている。そこだけ見ると問題がなさそうに見えるので、バグを見つけるのは困難。

  イテレータが複数あるときには、インデックスも明確な名前をつけるとよい。i, j, kではなく`club_i`、`members_i`, `users_i`など。もっと簡潔に、`ci`、`mi`,`ui`でもよい。こうすることでバグが見つけやすくなる。

  ```python
  if (clubs[ci].members[ui] == users[mi]) ## 最初の文字が一致していないからおかしい！
  if (clubs[ci].members[mi] == users[ui]) ## 最初の文字が一致しているから問題ない！
  ```

**アドバイス：**

> tmp・it・retvalのような汎用的な名前を使うときは、それ相応の理由を用意しよう。

### 抽象的な名前よりも具体的な名前を使う

変数や関数などの構成要素の名前は、抽象的ではなく具体的なものにしよう。

例えば、`ServerCanStart()`という名前のTCP/IPポートをサーバがリッスンできるか確認するメソッドがあったとする。この名前はちょっと抽象的。`CanListenOnPort()`のように、メソッドの動作をそのまま表す命名の方がより具体的でわかりやすくなる。



### 名前に情報を追加する

前述のとおり、名前は短いコメントのようなもの。絶対に知らせないといけない大切な情報があれば、「単語」を変数名に追加するようにする。

#### 値の単位

時間やバイト数のように計測できるものであれば、変数名に単位を入れるとよい。

- 単位のない仮引数と単位を追加したよりよい名前の仮引数の例
  - `Start(int delay)`: delay -> delay_secs ## 秒
  - `CreateCache(int size)`: size -> size_mb ## メガバイト
  - `ThrottleDownload(float limit)`: limit -> max_kbps ## kbps
  - `Rorate(float angle)`: angle -> degress_cw ## 度

#### その他の重要な属性を追加する

変数名に追加すべき情報として、単位以外にも「危険や注意を喚起する情報」もある。

- 情報を変数名に追加したほうが良い例

  | 状況                                                       | 変数名   | 改善後             |
  | ---------------------------------------------------------- | -------- | ------------------ |
  | passwordはプレーンテキストなので、処理する前に暗号化すべき | password | plaintext_password |
  | ユーザが入力したcommentは表示する前にエスケープが必要      | comment  | unescaped_comment  |
  | htmlの文字コードをUTF-8に変えた                            | html     | html_utf9          |
  | 入力されたdataをURLエンコードした                          | data     | data_urlenc        |

  全ての変数名に属性を追加する必要はない。変数の意味を間違えたときにバグになりそうなところにだけ使うことが大切。

  特にセキュリティ系のバグのような深刻な被害が出るところに使うとよい。



### 名前の長さを決める

いい名前を選ぶ時には、「長い名前を避ける」という暗黙的な制約がある。

長すぎても短すぎてもよくない。ではどのあたりに線を引けばよい？それは変数の使い方によって異なる。

#### スコープが小さければ短い名前でもいい

識別子の「スコープ」(その名前が「見える」コードの行数)が小さければ多くの情報を詰め込む必要はない。

```c++
if (debug) {
	map<string,int> m;
	LookUpNamesNumbers(&m);
	Print(m);
}
```

`m`という変数名には必要最低限の情報しか含まれていないが、コードを理解するのに必要な情報はそのすぐ下1行を見れば完結している。このような場合は`m`で問題ない。

`m`がクラスのメンバ変数やグローバル変数の場合

```javascript
LookUpNamesNumbers(&m);
Print(m);
```

このような場合、`m`の型や目的が不明で読みにくいコードである。識別子のスコープが大きければ、名前に十分な情報を詰め込み明確にする必要がある。



#### 長い名前を入力するのは問題じゃない

たいていのテキストエディタには補完機能があるため、少々名前が長くなってしまっても問題ない。



#### 頭文字と省略形

頭文字や省略形を使って名前を短くすることがある。省略可否の判断はどうすればよいか？

**「新しいチームメイト(将来の自分自身も)がその名前の意味を理解できるかどうか？」**を判断基準にすればまず問題ない。

例えば、`BackEndManager`じゃなくて`BEManager`にする。この場合、`BE = BackEnd` と理解できるのは恐らくプロジェクトメンバのみ。新しいチームメイトは理解できないだろう。プログラマ共通の語、例えば、`evaluation`の代わりに`eval`、`document`の代わりに`doc`、`string`の代わりに`str`のようなものは省略しても良い。

#### 不要な単語は投げ捨てる

名前に含まれる単語を削除しても情報が全く損なわれないこともある。その時は短くしてしまおう。例えば、`ConvertToString()`を短くして`ToString()`にしても必要な情報は何も損なわれない。

#### 名前のフォーマットで情報を伝える

アンダースコア・ダッシュ・大文字を使って名前に情報も詰め込むこともできる。

プロジェクトや言語によって使うフォーマット規約は違ってくる(JavaScriptなら『JavaScript: The Good Parts』、PythonならPEP8など)。規約を使うかどうかは、自分自身やチームで決めるとよい。どんなものを使うにしても、プロジェクトで一貫性を持たせることが大切。

## 誤解されない名前

**カギとなる考え：**

> 名前が「ほかの意味と間違えられることはないだろうか？」と何度も自問自答する。

- 例: filter()

  `filter`は曖昧。「選択する」という意味？「除外する」という意味？前者なら、`select()`、後者なら`exclude()`を使おう。

### 限界地を含めるときはminとmaxを使う

**アドバイス：**

> 限界地を明確にするには、名前の前に`max_`や`min_`をつけよう。

- 例：

  `CART_TOO_BIG_LIMIT=10`: 未満なのか、以下なのか曖昧。

  `MAX_ITEMS_IN_CART= 10`: こうすることで「item数の最大は10」ということが一目でわかる。



### 範囲を指定するときはfirstとlastを使う

- `start/stop`よりも`first/last`の方が明確

### ブール値の名前

ブール値の変数やブール値を返す関数の名前を選ぶときは`true/false`の意味を明確にしなければならない。

- 危険な例

  ```c++
  bool read_password = true;
  ```

  このような命名の場合、`read`には以下の二つの解釈ができてしまう

  - パスワードを**これから**読み取る必要がある
  - パスワードを**すでに**読み取っている

ブール値の変数名は、頭に`is・has・can・should`などをつけるとわかりやすくなる。

また、名前には否定形を避けるようにする。例えば、`disable_ssl = false;`ではなく、`use_ssl = true;`と肯定形にした方が直感的かつ、短くて済む。



### ユーザの期待に合わせる

たとえ別の意味で使っていたとしても、ユーザが先入観を持っているために誤解を招いてしまうことがある。潔く「負けを認めて」誤解されない名前に変えたほうが良い。

- 例: `get()`

  `get`で始まるメソッドはメンバの値を返すだけの「軽量アクセサ」であるという規約に慣れ親しんでいる。そのため、それ以上の動作を行うメソッドの場合には`get`という単語を避けるべき

### 複数の名前を検討する

名前を決める際は、複数の候補を検討しておき、それぞれの長所について話し合ってから最終判断を行うのがベスト。

話し合いの際には以下の項目を検討する。

- この機能を知らない人が見たらどうなるか想像し、
- どのような誤解が生まれるか反対意見を考える。
- 何が起きるかを明確に表していて、誤解を生む可能性が低いものを採用する。



## 美しさ

優れたソースコードは「目に優しい」ものでなければいけない。

著者が使っている3つの原則

- 読み手が慣れているパターンと一貫性のあるレイアウトを使う。
- 似ているコードは似ているように見せる。
- 関連するコードをまとめてブロックにする。

### なぜ美しさが大事？

プログラミングの時間のほとんどはコードを読む時間！さっと流し読みができれば、だれにとっても使いやすいコードだと言える。

### 一貫性のある簡潔な改行位置

「美しくない」例と「美しい」例を眺めてみる。

- 美しくない例

  ```java
  public class PerformanceTester{
  	public static final TcpConnectionsSimulator wifi = new TcpCoonectionsSimulator(
  		500, /* Kbps */
  		80, /* millisecs latency */
  		200, /* jitter */
  		1 /* packet loss %*/ );
  	public static final TcpConnectionsSimulator t3_fiber =
  	  new TcpCoonectionsSimulator(
  		45000, /* Kbps */
  		10, /* millisecs latency */
  		0, /* jitter */
  		0 /* packet loss %*/ );
  }
  ```

  美しくない要素：

  - 横幅80文字に合わせるために余計な改行が入っている
  - 同じコメントの繰り返し

- 美しく直した例

  ```java
  public class PerformanceTester{
    // TcpConnectionSimulator(throughput, latenct, jitter, packet_loss)
    //                            [Kbps]    [ms]    [ms]     [percent]
    
    public static final TcpConnectionSimulator wifi =
      new TcpConnectionSimulator(500, 80, 200, 1);
    public static final TcpConnectionSimulator t3_fiber =
      new TcpConnectionSimulator(45000, 10, 0, 0);
  ```

  コメントを最上部に移動して、仮引数を一行で書くようにした。数値の右隣からコメントは省略し、より簡潔な表組に「データ」が並ぶようになった。



### メソッドを使った整列

同じ動作を何度も繰り返すときはその動作をメソッドにしてしまい、別のコードブロックで書くことでコードの見た目を良くすることができる。

- 例: `CheckfullName`メソッドを定義

  ```python
  def CheckFullName(partial_name, expected_full_name, expected_error):
  	full_name = ExpandFullName(database_connection, partial_name, &error)
  	assert(error == expected_error)
  	assert(full_name == expected_full_name)
  
  CheckFullName("Doug Adams", "Mr. Douglas Adams", "")
  CheckFullName("John", "", "more than one result")
  ```

  コードの見た目が美しくなる以外にも、嬉しい副作用として、

  - 重複排除による簡潔化
  - テストケースの重要な情報(名前やエラー文字列)が見やすくなる
  - テストの追加が簡単になる

  のような改善もできる。

### 縦の線をまっすぐにする

スペースを適宜用いることで文章に目を通しやすくなる。

```python
CheckFullName("Doug Adams", "Mr. Douglas Adams", "")
CheckFullName("John"      , ""                 , "more than one result")
```

```python
details  = request.POST.get('details')
location = request.POST.get('location')
phone    = equest.POST.get('phone')
email    = request.POST.get('email')
```

整列しておくと下スニペットの3行目のようにタイポにも気づきやすくなる。

このTipsは好き嫌いが分かれるかもしれない。1行だけ変更したいのに、他の行も空白調整をしなければいけないため。試しにやってみて手間だと感じるようなら好きにやめればいい。



### 一貫性と意味のある並び

コードの並びがコードの動作に影響を及ぼすことは少ない。例えば、以下はどんな順番でも問題ない。

```python
details  = request.POST.get('details')
location = request.POST.get('location')
phone    = equest.POST.get('phone')
email    = request.POST.get('email')
```

であれば、ランダムに並べるのではなく、意味のある順番に並べるといい。順番を考えるときの項目として、

- 対応するHTMLフォームの<input>フィールドと同じ並び順にする

- 「最重要」なものから重要度順にする
- アルファベット順にする

などが考えられる。どの並べ順にするとしても一連のコードでは同じ並び順を使うべき。



### 宣言をブロックにまとめる

コードの概要を素早く把握してもらうにはグループや階層を１つの「単位」にまとめてしまえばよい。

```c++
class FrontendServer{
	public:
		FrontendServer();
		~FrontendServer();
		
		// ハンドラ
		void ViewProfile(HttpRequest* request);
		void SaveProfile(HttpRequest* request);
		void FindFriends(HttpRequest* request);
		
		//リクエストとリプライのユーティリティ
		string ExtractQueryParam(HttpRequest* request, string param);
		・・・
		
		// データベースのヘルパー
		void OpenDatabase(string location, string user);
		・・・
}
```

コード行は増えてしまうが、ずっと読みやすくなる。



### コードを「段落」に分割する

文章は複数の段落に分割されている。コードも同様に「段落」に分けるべき。

いくつかの手順に分割して書くとコードがすっきりする。

```python
def suggest_new_friends(user, email_password):
	# ユーザの友達のメールアドレスを取得
	friends = user.friends()
	friend_emails = set(f.email for f in friends)
	
	# ユーザのメールアカウントからすべてのメールアドレスをインポート
	contacts = import_contacts(user.email, email_password)
	・・・
	
	# まだ友達になっていないユーザを探す
	non_friend_emails = contact_emails - friend_emails
	・・・
	
	# ページに表示する
	・・・
	
	return render("suggested_friends.html", display)
```

段落ごとにようやくコメントを追加するとさらにコードを俯瞰しやすくなる。



### 個人的な好みと一貫性

最終的には個人の好みになってしまうこともある。ある程度整っていればどの書き方を選んでもコードの読みやすさに大きな影響はない。だが、複数のスタイルを混ぜてしまうと、すごく読みにくくなってしまう。プロジェクトの規約に従うようにして、一貫性を保つ方が大切。

**カギとなる考え：**

> 一貫性のあるスタイルは「正しい」スタイルよりも大切だ。