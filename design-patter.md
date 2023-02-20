# Template Method: アルゴリズムを変更する
## 概要
- 抽象クラスで骨格となるメソッドを定義し、実体はサブクラスごとに実装する。
- 変わらないもの(テンプレートメソッドに記述された基本的なアルゴリズム)と変わるもの(サブクラスで提供される詳細な処理)を分離できる
## コード例
```ruby
class Report
  def output_report
    output_start
    output_title(@title)
    output_body_start
    @text.each do |line|
      output_line(line)
    end
    output_body_end
    output_end
  end

  def output_start
    # デフォルトの動作を記述
  end
end

class PlainTextReport < Report
  def output_start
  end

  def output_head
    puts("**** #{@title} ****")
  end
end
```
## 注意点
- すべての抽象メソッドとフックメソッドには存在理由があるべき
  - サブクラスに大量のメソッドをオーバーライドさせるようなテンプレートクラスをは作成しない
- 継承にはデメリットもある
  - サブクラスがスーパークラスに依存する
  - 多重継承が(Rubyでは)できない
## まとめ
- アルゴリズムに多様性を持たせたいときに有用
# Strategy: アルゴリズムを交換する
## 概要
- Template Methodのようにバリエーションごとにサブクラスを作るのではなく、変化自体をコードに閉じ込めて分離する
## コード例
```ruby
class Report
  attr_reader :title, :text
  attr_accessor :formatter

  def initialize(formatter)
    @title = '月次報告'
    @text = ['順調', '最高の調子']
    @formatter = formatter
  end

  def output_report
    @formatter.output_report(self) # selfを渡すと結合度が上がる。必要な値を引数で渡すと引数の数が増える
  end
end

class PlainTextFormatter
  def output_report(context)
    puts "***** #{context.title} *****"
    context.text.each do |line|
      puts line
    end
  end
end
```
## 注意点
- ストラテジを呼ぶ側をコンテキストと呼ぶ
- コンテキストとストラテジの間でデータを共有するには、引数で渡すか、コンテキストのselfを渡す
- コンテキストとストラテジの間のインターフェースに注意する必要がある
  - 特定のストラテジとの依存度を強くすると、別のストラテジと入れ替えが効かず意味がない
- Rubyではストラテジとしてわざわざクラスを作らなくても、単純なものならブロックで実現できる
## まとめ
- StrategyはTemplate Methodの委譲ベースバージョン
# Observer: 変更に追従する
## 概要
- 「何らかのオブジェクトが変化した」というニュースの発信者と消費者の間に綺麗なインターフェースを作る
- サブジェクトとなるオブジェクトが内部にobserversというフィールドを持つ
- それらにobserverを追加し、通知を送信する
## コード例
```ruby
module Subject
  def initialize
    @observers = []
  end

  def add_observer(observer) # &observerにしてコードブロックをオブザーバーとして渡すことも可能
    @observers << observer
  end

  def delete_observer(observer)
    @observers.delete(observer)
  end

  def notify_observers
    @observers.each do |observer|
      observer.update(self)
      # observer.call(self)
    end
  end
end

class Employee
  include Subject

  attr_reader :name, :address
  attr_reader :salary

  def initialize(name, title, salary)
    super()
    @name = name
    @title = title
    @salaly = salary
  end

  def salaly=(new_salary)
    @salaly = new_salary
    notify_observers
  end
end

class TaxMan
  def update(changed_employee)
    puts "#{changed_employee.name}に新しい税金の請求書を送ります"
  end
end

fred = Employee.new('Fred', 'Crane Operator', 30000)
fred.add_observer(TaxMan.new)
# fred.add_observer do |changed_employee|
#   puts('changed!')
# end
```
## 注意点
- サブジェクトとオブザーバー間のインターフェースが重要
  - pull型: オブザーバーには一つのメソッド(update)があり、引数としてsubjectのselfを渡す
    - オブザーバーがsubjectから色々と情報を引き出す必要がある
    - オブザーバーの仕事 > subjectの仕事
  - push型: オブザーバーに複数のメソッド(update_name, update_salary)があり、引数として値を渡す
    - オブザーバーの仕事は減る
    - 関心がない値を渡されるオブザーバーがいる場合、無駄になる
- 更新の頻度とタイミングが重要
  - 毎秒数千回の更新をオブジェクトが行うかもしれない
  - 通知を送信するタイミングを見極めないと、不整合な状態を聞いているobserverが存在するかも
  - オブザーバーが例外を出した場合どうする？
## まとめ
- Observerパターンは他のコンポーネントの動きを監視するコンポーネントが作れる
- オブジェクト同士を強結合することもない
- Strategyとの違い
  - Strategy: 何らかの処理を行う
  - Observer: オブザーバブルp武ジェクトで発生しているイベントを他のオブジェクトに伝える
- PubSubと似ているが、PubSubは間にメッセージブローカーがそんざいし、送信側も受信側もお互いを知らない
# Composite 部分から全体を組み立てる
## 概要
- 大きなオブジェクトが小さい子オブジェクトから構成されている状態(階層構造・ツリー構造)
- あるオブジェクトに対する操作を、一つのノードを扱っているのか、枝全体を扱っているのかを意識せずに行えるようにする
- Component: 全てのオブジェクトの共通のインターフェースまたは基底クラス
- Leaf: 単純な構成要素で、Componentインターフェースも実装している
- Composite: いくつかの子から構成される
## コード例
```ruby
class Task # Component
  attr_reader :name, :parent

  def initialize(name)
    @name = name
  end

  def get_time_required
    0.0
  end
end

class MixTask < Task # Leaf
  def initialize
    super('Mix that batter up!')
  end

  def get_time_required
    3.0
  end
end

class CompositeTask < Task # Composite(の基底クラス)
  def initialize(name)
    super(name)
    @sub_tasks = []
  end

  def <<(task)
    @sub_tasks << task
    task.parent = self # 親をたどれるようにもできる
  end

  def remove(task)
    @sub_tasks.delete(task)
    task.parent = nil
  end

  def get_time_required
    time = 0.0
    @sub_tasks.each { |task| time += task.get_time_required }
    time
  end
end

class MakeBatterTask < CompositeTask # LeafとCompositeからなるComposite
  def initialize
    super('Make batter')
    self << AddDryIngredientsTask.new
    self << MixTask.new
  end
end
```
## 注意点
- CompositeとLeafを同列に扱いたいが、避けられない違いがある
  - Leafは子がいない
    - Leafに対しても子を追加するメソッド(何もしない)を持たせる？
    - Leafに子を扱うメソッドがないのを受け入れる(おすすめ)
- ツリーの深さが1段とは限らないことに注意
## まとめ
- 個々のコンポーネントからなる複雑なオブジェクトが、子の性質と特徴を共有しているときに使える
  - GUIライブラリなど
- IteratorパターンはCompositeパターンを特化したもの
# Iterator: コレクションを操作する
## 概要
- 集約オブジェクトが元にある内部表現を公開せずに、その要素に順にアクセスする方法を提供する
- 外部イテレータ
  - イテレータオブジェクトを提供する
  - 外部イテレータでn番目の要素を取り出して、それに対して何か処理する
- 内部イテレータ
  - 集約オブジェクトの内部で繰り返し処理が起こる
  - コードブロックベースのイテレータ
## コード例
```ruby
class ArrayIterator # 外部イテレータ
  def initialize(array)
    @array = array
    @index = 0
  end

  def has_next?
    @index < array.length
  end

  def item
    @array[@index]
  end

  def next_item
    value = @array[@index]
    @index += 1
    value
  end
end

array = ['red', 'green', 'blue']
i = ArrayIterator.new(array)
while i.has_next?
  puts("item: #{i.next_item}")
end

# IOオブジェクトも外部イテレータの一つ
f = File.open('names.txt')
while not f.eof?
  puts f.readline
end
```
## 注意点
- リストを走査している間にリスト事態に変更が行われた場合の動き
  - マルチスレッドでは特に注意
  - コピーを作ることで回避は可能
## まとめ
- 外部イテレータ
  - コレクションのメンバーを指し示すオブジェクト
- 内部イテレータ
  - 何らかのポインタを渡す代わりに、子オブジェクトを扱うためのコードを渡す
- Rubyのイテレータは優秀
# Command: 命令を実行する
## 概要
- 動作用のコードをオブジェクトに抜き出す
- 「動作の定義」と「動作の実行」を分離する
  - 「これを行え」の代わりに、「これを行う方法を記録しろ」「記録したことを行え」
- execute(とunexecute)のみをもったクラス
## コード例
```ruby
class Comand
  attr_reader :description

  def initialize(description)
    @description = description
  end

  def execute
  end
end

class CompositeCommand < Command # Compositeパターン
  def initialize
    @commands = []
  end

  def <<(cmd)
    @commands << cmd
  end

  def execute
    @commands.each { |cmd| cmd.execute }
  end

  def unexecute
    @commands.reverse.each { |cmd| cmd.unexecute }
  end

  def description
    description = ''
    @commands.each { |cmd| description += cmd.description + "\n" }
    description
  end
end

class DeleteFile < Command
  def initialize(path)
    super "Delete file: #{path}"
    @path = path
  end

  def execute
    if File.exists?(@path)
      @contents = File.read(@path) # 可逆性を持たせるためにコピーを取っておく
    end
    f = File.delete(@path)
  end

  def unexecute
    if @contents
      f = File.open(@path, "w")
      f.write(@contents)
      f.close
    end
  end
end
```
## 注意点
- 本当にこのパターンを使う必要があるのかを検討
- 何を行うかの決定と実行時の間に環境や前提条件が変化した場合を考慮する必要あり
## まとめ
- ActiveRecordのマイグレーションがいい例
- これから行うことのリストや、完了したことのリストを記録する必要のあるときに適する
- 行ったコマンドを元に戻すこともできる
- クラスで実装すると複雑だが、rubyだとブロックで実装する手もある
- CommandとObserverの違い
  - Command: 何かを行う方法を知っているだけで、実行する対象の状態には関心がない
  - Observer: 呼び出される対象の状態に関心がある
# Adapter: ギャップを埋める
## 概要
- インターフェイス間の不整合のギャップを埋める
## コード例
```ruby
class Encrypter
  def initialize(key)
    @key = key
  end

  def encrypt(reader, writer)
    key_index = 0
    while not reader.eof?
      plain_char = reader.getc # readerにはIOオブジェクト(getcのインターフェイス)を期待している
      encrypted_char = plain_char ^ @key[key_index] # XOR演算で暗号化
      writer.putc(encrypted_char)
      key_index = [key_index + 1] % @key.size
    end
  end
end

class StringIOAdapter
  def initialize(string)
    @string = string
    @position = 0
  end

  def getc
    if @position >= @string.length
     raise EOFError
    end

    cd = @string[@position]
    @position += 1
    ch
  end

  def eof?
    @position >+ @string.length
  end
end

Encrypter = Encrypter.new('XYZZY')
reader = StringIOAdapter.new('We attack at dawn') # Clientが期待するインターフェイスを備えたAdapter
writer = File.open('out.txt', 'w')
encrypter.encrypt(reader, writer) # Client(encrypter)は対象のオブジェクトがアダプタであるとは知らない
```
## 注意点
- Rubyではオープンクラスなどでアダプタなしに直接インターフェイスを変更することが可能
  - 変更内容がシンプルで明快ならあり
  - インターフェイスの不整合が広範囲で複雑な場合はAdapterで解決する方が良い
- 必要だと思わなかったメソッド(上記で言うと`getc`と`eof?`以外)が呼ばれると動かない
## まとめ
- 必要なインターフェイスと既存のオブジェクトとの間の違いを吸収するためにある
  - 外部には必要なインターフェイスを提供し、内部に隠されたオブジェクトを持つ
- ProxyやDecoratorパターンが仲間
# Proxy: オブジェクトに代理を立てる
## 概要
- 以下のような課題を解決する
  - オブジェクトへのアクセス制限
  - 場所に依存しないオブジェクトの取得
  - オブジェクト生成の遅延
## コード例
### 基本のProxy
```ruby
class BankAccount
  attr_reader :balance

  def initialize(starting_balance = 0)
    @balance = starting_balance
  end

  def deposit(amount)
    @balance += amount
  end

  def withdraw(amount)
    @balance -= amount
  end
end

class BankAccountProxy # 現状はただ委譲しているだけだが、クライアントと本物のオブジェクトの間に立ち入る場所になる
  def initialize(real_object)
    @real_object = real_object
  end

  def balance
    @real_object.balance
  end
  ...
end

proxy = BancAccountProxy.new(BankAccount.new(100))
proxy.deposit(50)
proxy.withdraw(10)
```
### 防御Proxy
- 検証ロジックを分離できる = 関心事の分離ができる
```ruby
require 'etc' # 現在のユーザー名を取得するモジュール

class AccountProtectProxy
  def initialize(real_account, owner_name)
    @subject = real_account
    @owner_name = owner_name
  end

  def deposit(amount)
    check_access
    @subject.deposit(amount)
  end

  private

  def check_access
    if Etc.getlogin != @owner_name
      raise "Illegal access: #{Etc.getlogin} cannot access account"
    end
  end
end
```
### リモートProxy
- ネットワークへの関心と、天気情報の取得への関心を切り離せる
```ruby
require 'soap/wsdlDriver'

wsdl_url = 'http://www.example.com'
proxy = SOAP::WSDLDriverFactory.new(wsdl_url).create_rpc_driver
# Proxyを用いる事で、RPCのネットワークへの関心と、天気情報の取得への関心を切り離せる
# Proxyを入れ替えれば通信プロトコルのみ入れ替えることも容易
weather_info = proxy.GetWeatherByZipCode('ZipCode' => '19128')
```
### 仮想Proxy
- 作成コストがかかるオブジェクトの生成を本当に必要になるまで遅延できる
- インスタンス生成時の問題と、実際の預入や引き出し操作の関心ごとを分離する
```ruby
class VirtualAccountProxy
  def initialize(&creation_block) # starting_balanceを引数に渡すとオブジェクト生成の責務も持ってしまう
    @creation_block = creation_block
    # @starting_balance = starting_balance
  end

  def deposit(amount)
    subject.deposit(amount)
  end

  private

  def subject
    @subject ||= @creation_block.call
    # @subject ||= BankAccount.new(@starting_balance)
  end
end
```
## 注意点
- 委譲するメソッドが多い時の対応
  - method_missingで一括対応もできるが、全てのオブジェクトが持っているメソッドは捕捉されない
  - method_missingは継承ツリーを全て辿るので少し遅い
## まとめ
- あるオブジェクトが別のオブジェクトの代わりをするパターンの一つ
- Adapterがインターフェースを変換するのに対し、Proxyは内部のオブジェクトへのアクセスを制御する