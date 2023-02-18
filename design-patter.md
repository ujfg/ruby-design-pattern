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