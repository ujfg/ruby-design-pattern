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