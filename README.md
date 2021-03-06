# P2279R0 Review
[P2279R0: We need a language mechanism for customization points](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2279r0.html)  
2021-01-15 Barry Revzin  

C++のインターフェイスを「適切にカスタマイズする」ための言語サポートが欲しいよね。  
いま議論の俎上に載せることができそうな話題としては、以下のような方法がありそうです。

- virtual member functions
- Existing Static Polymorphism Strategies
	- Class Template Specialization
	- ADL
		- "pure" ADL
		- Customization Point Objects (aka CPOs)  
			- [N4381: Suggested Design for Customization Points](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4381.html)  
		   　　2015-03-11 Eric Niebler
		- tag_invoke
			- [P1895: tag_invoke: A general pattern for supportingcustomisable functions](http://open-std.org/JTC1/SC22/WG21/docs/papers/2019/p1895r0.pdf)  
		   　　2019-10-07 Lewis Baker, Eric Niebler, Kirk Shoop
- Relevant Work
	- Customization Point Functions
	- Reflective Metaprogramming
	- C++0x Concepts

それぞれ一長一短 (??) で、色々な特徴や問題点もあるので、公式な言語機能があって欲しい。  
この paper が良い出発点になれば。議論をスタートできるといいなと。

参考資料: [(地面を見下ろす少年の足蹴にされる私)](https://onihusube.hatenablog.com/#P2279R0-We-need-a-language-mechanism-for-customization-points)

# 目次
- Customization Point Objects (CPO)
- C++0x Concept Maps (concept_map)
- 参考資料
- 実装の研究 - CPO
	-  GCC libstdc++-v3
	-  LLVM libcxx
	-  MS STL

# Customization Point Objects (CPO)
C++20 で導入された新しいデザインパターン。[\[customization.point.object\]](https://timsong-cpp.github.io/cppwp/n4861/customization.point.object)  

semiregular な関数オブジェクト。 ( callable function object )  
「制約のある ADL (constrained ADL dispatch)」を行うために存在する。 
```cpp
// 関数オブジェクト
struct functor {
  void operator()(int i) { std::cout << i << std::endl; }
};
```

- 提案
	- [N4381: Suggested Design for Customization Points](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4381.html)  
		- 2015-03-11 Eric Niebler
	- [Customization Point Design in C++11 and Beyond](http://ericniebler.com/2014/10/21/customization-point-design-in-c11-and-beyond/)
		- 2014-10-21 Eric Niebler

関数オブジェクトにて C++20 Concept による必要なチェックをした後、(`model`)  
ADL が有効な文脈に実引数を渡すことによって、ADL を制御下に置くことができるようになる。

ADL はそのままだと制御が簡単ではなく、( 意図しない動作やエラー等, 若干制御不能. )  
もし意図しない適用が起きていたとしても検出できるのは実行時だったり難しい。

CPO は、ADL を制御しようとする試みです。 ( `niebloid`を実現する実装方法の 1つ )  
C++23 検討中の Executor ライブラリは、Concept と CPO ベースの非常にジェネリックな最先端のライブラリになっている。

- 用語
	- ADL : Argument-dependent name lookup
	- model : [Concept](https://cpprefjp.github.io/reference/concepts.html) に適合すること。また適合している型(の集合)。
	- niebloid : ADL を妨げることができるアルゴリズム全般のこと。

## ___customization point___
独自のユーザ定義実装に変更 (カスタマイズ) することができる、C++標準ライブラリ (STL) の関数。 - [N4381](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4381.html)
- C++標準ライブラリの関数で、user’s namespace にある user-defined types によって  
オーバーロードすることができ、かつ、ADL によって見つかるもの。 

C++標準ライブラリには、ユーザ側で挙動を変更できる箇所がすでにいくつか存在していた。( C++11 )
- `std::swap`
- `std::begin`
- `std::end` etc.

[N4381](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4381.html) では、
1. customization point をユーザ定義する際の、現行アプローチの使い勝手の問題点の記述
2. 将来の customization point 定義において利用できるデザインパターンの提示

を行うこととしている。

### 現行アプローチ
```cpp
// イテレート可能な範囲を受けて何かする関数
template<typename Container>
void func(Container&& c) {
  using std::begin;
  using std::end;

  auto itr = begin(c);
  auto end = end(c);
}
```
- `std::begin`, `std::end` という名前を name lookup で発見できるようにする。 (`using`)
- `begin()`, `end()` を名前空間修飾なしで呼び出す。  
  → unqualified name lookup を行うと ADL も働き、見つかった名前の中でオーバーロード解決が行われる。  
	- `std`名前空間のもの 及び 配列 には、`std::begin()`, `std::end()` が呼び出されるようになる。
	- ユーザ定義型に対しては、同じ名前空間内にある `begin()`, `end()` あるいは  
	  `std::begin()`, `std::end()` を通してメンバ関数の `begin()`, `end()` が呼び出されるようになる。

#### 背景 - ジェネリック:「標準 begin/end」と「ユーザ定義 begin/end」を合わせて扱う
```cpp
// 独自のユーザ定義型
struct X { int data[100]; };

// 独自のbegin/end実装
//  → 独自型の名前空間スコープに, 同名の関数/関数テンプレートを書けばよい. ADLで適切な関数が選ばれる.
int* begin(X& x) { return data; }
int* end(X& x) { return data + 100; }
```

### 問題点 2つ
1. ___error-prone___ : 誤って利用されるリスクがある ( 呼び出しが煩雑, 使いづらい )
	- `using` とか。ADL とか。原理の説明に込み入った知識を要求するし、誤使用されやすい。
		- `using` 無しで `std::begin(c)`, `std::end(c)` とすると (下記)、ユーザ定義実装が呼ばれないことが起きる。
		- ( `using` しないと `std::begin()`, `std::end()` が見つからないことが起きる ※ )
			- [関数テンプレート特殊化とADLの小改善](https://yohhoy.hatenadiary.jp/entry/20181215/p1) (yohhoyの日記)
	- 正しい呼び出し方法が煩雑で、C++ を深めに理解する事が求められるなど、使いづらい。 
```cpp
template<typename Container>
void func(Container&& c) {
  auto itr = std::begin(c);
  auto end = std::end(c);
}
```
2. ___constraint bypass___ : 要求される型制約を無視できてしまう ( コンセプトを用いた型制約を強制できない )
	- 標準ライブラリの `std` 側で C++20 Concept による制約をしても、  
	  ADL 経由で制約を満たさない `begin()` が呼ばれることがある。  
	  → コンセプトによる制約を簡単に迂回できてしまう。
```cpp
// イテレータを返してくれない俺俺ユーザ定義...
//   → 名前が同じで, 全く異なる意味を持つ関数が定義されていると未定義動作に陥る可能性がある.
bool begin(X& x) { return true; }
```

### 設計のゴール (Design Goals)
- 完全修飾名呼び出し (qualified lookup) `std::begin(c);`、または  
  非修飾名呼び出し (unqualified lookup) `using std::begin; begin(c);` は、  
  いずれの呼び出しであっても同じ振る舞いになるようにすること。  
  (そして) 特に、ユーザ定義オーバーロードを引数の名前空間から見つけ出せること。 
- `using std::begin; begin(c);` としても、CPO の `std::begin` が要求する型制約がバイパス (無視, 迂回) されないこと。
	- 関数オブジェクトである事によって防止される。
 
C++20 では、C++標準ライブラリ への CPO 導入によって、既存の 2つ の課題解決をはかっている。  
(※ 上記、単にわかりやすさの為だけに CPO を `begin` で書いてしまっています. お許しを.)  

実際には C++20 では `std::ranges::begin()` で解決している。

### 設計の詳細 (Design Details)
#### [N4381](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4381.html) が提案していること
```cpp
namespace std {
  namespace __detail {
  
    // define begin for arrays
    template <class T, size_t N>
    constexpr T* begin(T (&a)[N]) noexcept {
      return a;
    }

    // Define begin for containers
    // (trailing return type needed for SFINAE)
    template <class _RangeLike>
    constexpr auto begin(_RangeLike && rng) ->
      decltype(forward<_RangeLike>(rng).begin()) {
      return forward<_RangeLike>(rng).begin();
    }

    struct __begin_fn {
      template <class R>
      constexpr auto operator()(R && rng) const
        noexcept(noexcept(begin(forward<R>(rng)))) ->
        decltype(begin(forward<R>(rng))) {
        return begin(forward<R>(rng));  // ★2: constrained ADL dispatch
      }
    };
    
  } // namespace __detail

  // To avoid ODR violations:
  template <class T>
  constexpr T __static_const{};

  // std::begin is a global function object
  namespace {
    constexpr auto const & begin =
        __static_const<__detail::__begin_fn>;  // ★1: 関数オブジェクト (inhibit ADL)
  }
}
```
#### std::ranges::begin が定義していること

　( まだ. )
 
 文末の「実装の研究」にリンクがある、実際の実装も参照ください。

#### 関数オブジェクトと ADL について
- [\[basic.lookup.argdep\] 3](https://timsong-cpp.github.io/cppwp/n4861/basic.lookup.argdep#3)  
	- unqualified lookup の結果として得られた集合 (lookup set X) に「関数オブジェクト」のような  
	  「 (本物の) 関数 や 関数テンプレート でないもの」が含まれていると ADL (lookup set Y) は一切適用されない.
		- → __関数オブジェクト に対して ADL は発動しない。__
		- → 必ず意図したものが呼ばれるようにコンパイルされるので、そこに何かのチェックを咬ませることが簡単にできる。
			- [What is a niebloid?](https://stackoverflow.com/questions/62928396/what-is-a-niebloid) (stackoverflow)
			- [ADLを無効にする関数定義](https://cpprefjp.github.io/article/lib/disable_adl_function.html) (cpprefjp) ※
	- [Argument-dependent name lookup \[basic.lookup.argdep\] (3.3)](https://timsong-cpp.github.io/cppwp/n4861/basic.lookup.argdep#3.3)
- [\[algorithms.requirements\] 2](https://timsong-cpp.github.io/cppwp/n4861/algorithms.requirements#2)
- [\[namespace.std\] 2](https://timsong-cpp.github.io/cppwp/n4861/namespace.std#7)
	- [\[namespace.std\] 173)](https://timsong-cpp.github.io/cppwp/n4861/namespace.std#footnote-173)

### 新 C++20 Ranges での記述
```cpp
// イテレート可能な範囲を受けて何かする関数
template<typename Container>
void func(Container&& c) {
  auto itr = std::ranges::begin(c);
  auto end = std::ranges::end(c);
}
```
- Rangeライブラリ ( C++20: N4861 [PDF](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/n4861.pdf)/[HTML](https://timsong-cpp.github.io/cppwp/n4861/) )
	- \<concepts\> - [18 Concepts library \[concepts\]](https://timsong-cpp.github.io/cppwp/n4861/#concepts)
		- `ranges::swap` [\[concept.swappable\]](https://timsong-cpp.github.io/cppwp/n4861/concept.swappable) 
	- \<ranges\> - [24 Ranges library \[ranges\]](https://timsong-cpp.github.io/cppwp/n4861/#ranges)
		- `ranges::begin` [\[range.access.begin\]](https://timsong-cpp.github.io/cppwp/n4861/range.access.begin)
		- `ranges::end` [\[range.access.end\]](https://timsong-cpp.github.io/cppwp/n4861/range.access.end) etc.
- 従来から customization point になっている関数は、( CPO に置き換えたかったが )  
  後方互換性維持のため C++17 ライブラリ仕様のまま維持される。
	- CPO は別の名前空間に同名で定義される。
- ( `std::range` が見えているときは、`find`, `distance` など従来の `std` の関数は  
  　ADL によって呼び出されないようにしている。 - niebloid ) ※

# C++0x Concept Maps (concept_map)

- __コンセプト__
	- テンプレート引数に対する要件 (使用できる関数、ネストした型など) を記述したもの。
		- 型を記述するメタな型。ジェネリック・プログラミングに (メタな) 型安全の概念が導入される。
- __コンセプトマップ__
	- 「コンセプト」と「実在する型」を関連づけるもの。
	- C++0x draft にのみ存在した。
		- ( 正式版 C++11/14/17/20 には存在しない. C++11に採択されず削除された. )
- __constrained template__
	- テンプレート仮引数に対するコンセプト要件を明示的に指定したテンプレート。
		- テンプレート実引数をコンセプトに適合するものだけに制限する。
			- 適合しなければ、いち早くエラーにできる。( lookupで探し続けたり列挙したりしない )
	- Concept は template argument を制御するもの。

下記は C++0x concept map template の一例となる "ひとつづき" のソースコード。 
```cpp
// C++0x Concept
//  - コンセプト "Stack" を定義する.
concept Stack< typename T >
{
  // ネストした型 value_type を備える.
  typename value_type; // 型名 value_type が使用可能であることを要件として表現.

  // 大域関数 push(), pop(), top(), empty() を備える.
  void       push ( T&, const value_type& );
  void       pop  ( T& );
  value_type top  ( const T& );
  bool       empty( const T& );
};
```
```cpp
// C++0x concept_map (concept map template)
//  - コンセプトマップはテンプレートにできる.
//  - コンセプト"Stack" を std::vecor<T> に適合させる.
template< typename T >
concept_map Stack< std::vector< T > >
{
  typedef T value_type; // 要件になっている型を定義.

  void push ( std::vector< T >& v, const T& x ) { v.push_back( x ); }
  void pop  ( std::vector< T >& v )             { v.pop_back(); }
  T    top  ( const std::vector< T >& v )       { return v.back(); }
  bool empty( const std::vector< T >& v )       { return v.empty(); }
};
```
```cpp
// C++0x constrained template
//  - 仮引数 T がコンセプト "Stack" に適合することを指定.
//      - requires Stack< T >
//      - template< Stack T > と書いても OK. 
//  - 仮引数 T に対する実引数は、コンセプト "Stack" に適合する型だけに強制できる.
template< typename T > requires Stack< T >
void func( T& t, T::value_type x )         // 要件となっている型 value_type が登場してます.
{
  push( t, x );
  push( t, x );
  push( t, x );
  std::cout << std::boolalpha << empty( t ) << std::endl;

  pop( t );
  pop( t );
  pop( t );
  std::cout << std::boolalpha << empty( t ) << std::endl;
};
```
```cpp
// concept_map によって std::vector<T> はコンセプト "Stack" を満足し、
// 関数テンプレート f() の第1引数として渡すことができるようになる.
int main()
{
  std::vector< int > v;
  func( v, 2011 );
}
``` 

ここでもし仮に、`concept Stack` に適合するデフォルトの constrained template の型定義や `func` が  
C++標準ライブラリにおいて定義・提供されている、として、  
さらに `std::vector<T>` は、仮に独自のユーザ定義型である、と置き換えて考えた場合、  
`push()`, `pop()`, `top()`, `empty()` は customization point になる。

C++0x concept_map によって、既に存在するオブジェクトの定義を変えることなく、  
customization point をユーザ定義にカスタマイズすることができる。 - わかりやすい ※

そしてそのとき、Concept が要求する型制約を強制することができ、また、  
適合しない場合は容易に ( 超わかりやすい ※ ) エラーにすることができる。

- おまけ: Concept Map が生まれた経緯
	- [What were C++0x concept maps?](https://isocpp.org/wiki/faq/cpp0x-concepts-history#cpp0x-concept-maps) (isocpp.org : C++0x Concepts — Historical FAQs)

### 本題 paper レビュー
[3.3 C++0x Concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2279r0.html#c0x-concepts) - P2279R0: We need a language mechanism for customization points
- C++0x Concepts は、Rust の `trait` が実現していることと同じ解決方法。特に違いは無い。記述が簡素。
	- ( customization point objects, tag_invoke, customization point functions は記述が多くなる )
- customization point objects, tag_invoke, customization point functions は、
	- 独立した customization point で切り替える。
	- 関係が近いことを示すには `concept` (C++20 Concept) でグループ化することになる。
- C++0x Concepts だと customization point が集まってる。
	- ただ、opt-in mechanism が少し違ってる。`concept_map` を使う。
	- invocation model も違う。callable じゃないので forward で転送できないっぽい。

# 参考資料
- 江添さんの解説
	- 2010-09-20 [コンセプトの経緯](https://cpplover.blogspot.com/2010/09/blog-post_8970.html)
	- 2015-05-13 [N4381: Suggested Design for Customization Points](https://cpplover.blogspot.com/2015/05/c2015-04-pre-lenexa-mailings-n4381-n4389.html)
		- C++11 `std::swap`, `std::begin`, `std::end` 
- 新しい言語規格ドラフトのレビュー・解説
	- [What are customization point objects and how to use them?](https://stackoverflow.com/questions/53495848/what-are-customization-point-objects-and-how-to-use-them) (stackoverflow)
	- [Customization pointについて](http://tk0xleader.blog.shinobi.jp/c--/%E3%80%90c--2a%E3%80%91customization%20point%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6) (tk-xleaderのブログ)
		- `using std::swap;` や、`using namespace std;` として `swap` を非修飾名で呼び出す方法
			- (探索先にstd名前空間を加えてADLを意図的に引き起こす手段)
	- 地面を見下ろす少年の足蹴にされる私
		- [カスタマイゼーションポイントオブジェクト (CPO) 概論](https://onihusube.hatenablog.com/entry/2020/06/26/225920) (2020-06-26)
		  > C++における名前探索では修飾名探索と非修飾名探索を行なった後、引数依存名前探索（ADL）を行いオーバーロード候補集合を決定します。この時、非修飾名探索の結果に関数以外のものが含まれているとADLは行われません。逆に言うと、ADLは関数名に対してしか行われません。つまり、関数オブジェクトに対してはADLは発動しません（[6.5.2 Argument-dependent name lookup \[basic.lookup.argdep\]](https://timsong-cpp.github.io/cppwp/n4861/basic.lookup.argdep#3.3)）
			- これまでのカスタマイゼーションポイント
				- 従来の CP関数 を CPO に置き換えたかったが、互換性の問題からできなかったようです
				- そのため、CPO は別の名前空間に同名で定義されています。
			- C++20のカスタマイゼーションポイントオブジェクト
				- [C++17 CPO, C++20 CPO の対応と一覧](https://onihusube.hatenablog.com/entry/2020/06/26/225920#C20%E3%81%AE%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%9D%E3%82%A4%E3%83%B3%E3%83%88%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88)
			- C++23以降の標準ライブラリ  
			  > 特にC++23に導入されるのがほぼ確実視されているExecutorライブラリは、現段階ですでにコンセプトとカスタマイゼーションポイントオブジェクトベースの非常にジェネリックな最先端のライブラリです。C++23とそれ以降の標準ライブラリではカスタマイゼーションポイントオブジェクトとコンセプトは空気のような存在になるでしょう。 
	 	- [std::rangesの範囲アクセス関数 (オブジェクト) の使いみち](https://onihusube.hatenablog.com/entry/2019/12/26/200203) (2019-12-26)  
	 		- Rangeライブラリ
	- yohhoyの日記
		- [Customization Point Object](https://yohhoy.hatenadiary.jp/entry/20190403/p1) (2019-04-03)  
		  > C++標準ライブラリへのCPO導入によって、既存の2つの課題解決をはかっている。   
		  > ...  
		  > WD N4810現在のC++2a標準ライブラリでは、Rangesライブラリ要素として下記CPOを導入する（名前空間stdは省略）。
		  > 後方互換性維持のため、従来`swap`, `begin`, `end`はC++17ライブラリ仕様のまま維持される。
		- [関数テンプレート特殊化とADLの小改善](https://yohhoy.hatenadiary.jp/entry/20181215/p1) (2018-12-15)
- C++言語規格の資料
	- [C++の名前解決](https://prettysoft.hatenablog.com/entry/20101128/1497356882) (プログラミングの教科書を置いておくところ)
	- [C++の名前解決 ADL その2](https://prettysoft.hatenablog.com/entry/20101129/204558) (プログラミングの教科書を置いておくところ)  
	  　　　　( Nxxxx は Working Draft の Update版 でした。 - 参考: [cpprefjp - C++国際標準規格](https://cpprefjp.github.io/international-standard.html) )
		- ADL と関数テンプレート : N1905, N4713
		- ADL と関数オブジェクト : N3000  
		  > 今まで特に注目されていなかったこの仕組みですが、C++20 では、この関数呼び出しのように見えるのに ADL が働かないという性質がカスタマイゼーションポイントオブジェクト（CPO）として利用されるようです 
		- ADL とカスタマイゼーションポイント : N3000, [N4741(2018)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/n4741.pdf)
		- ADL とカスタマイゼーションポイントオブジェクト(CPO) : [N4762(2018)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/n4762.pdf)
			- CPO というのは、ADL を制御しようとする試み
				- 関数呼び出し時に適用される ADL はそのままでは制御不能
				- もし意図しない適用が起きていたとしてもそれを検出できるのは実行時
			- ここで関数オブジェクトが注目されます
				- ___関数オブジェクトは本当の関数ではないので、ADL は適用されません___ ★★★
				- 必ず意図したものが呼ばれるようにコンパイルされるので、そこに何かチェックを咬ませることが簡単にできる
					- C++20 にはコンセプトが導入される
						- コンセプトで必要なチェックをした後に、ADL が有効な文脈に実引数を渡してあげると  
						  ADL を制御下に置くことができるようになる
			- ADL が制御不能であるというところは変わりませんが、  
			  その前にコンパイル時のチェックができるということで幾分安心できるようになりそう
		- niebloid (Eric Niebler)
			- [Customization Point Design in C++11 and Beyond](http://ericniebler.com/2014/10/21/customization-point-design-in-c11-and-beyond/)  - 2014-10-21  Eric Niebler
		- std::tag_invoke (C++23 で導入を目指している)
	- Barry Revzin's github
		- [Niebloids and Customization Point Objects](https://brevzin.github.io/c++/2020/12/19/cpo-niebloid/) (2020-12-19)
			- [std::ranges::swap (C++20)](https://cpprefjp.github.io/reference/concepts/swap.html) (cpprefjp)
			- [\[customization.point.object\]](http://eel.is/c++draft/customization.point.object#def:customization_point_object) (c++draft)  
			  > In the standard library, customization point object is a specific term of art that is a semiregular, function object that is const-invocable. And the standard library has a whole bunch of them in the `std::ranges` namespace (`begin`, `end`, `swap`, etc.) but even a few in plain old `std` (e.g. `strong_order`).
			- Niebloids solve undesired ADL
				- [\[algorithms.requirements\] 2](http://eel.is/c++draft/algorithms#requirements-2) (c++draft)
				- [P1292R0: Customization Point Functions](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1292r0.html)  - 2018-10-08 Matt Calabrese
				- [STL2 Issue #371](https://github.com/ericniebler/stl2/issues/371)
			- Niebloids vs Customization Point Objects
				> In short:  
				> - a ___customization point object___ is a semiregular, function object (by definition) that exists to handle constrained ADL dispatch for you. `ranges::begin`, `ranges::swap`, etc, are customization point objects.  
				> - a ___niebloid___ is a colloquial name for the algorithms declared in `std::ranges` that inhibit ADL. The only implementation strategy in standard C++20 for them is to make them function objects, but they are not required to be objects at all (nor are they required to be copyable, etc.) and a possible future language extension would allow them to be implemented as proper function templates.  
				> 
				> These terms are completely disjoint - ... 
		- [Declaratively implementing CPOs](https://brevzin.github.io/c++/2019/09/23/declarative-cpos/) (2019-09-23)  
		  > One of the new design patterns in C++20 is something known as a customization point object, or CPO. A CPO is a callable function object, which means you can easily pass it around to other functions without having to worry about the struggle that is passing around other kinds of polymorphic callables (like function templates and overload sets).  
		  > ...  
		  > One of the new CPOs from Ranges is `std::ranges::begin`, specified in [\[range.access.begin\]](http://eel.is/c++draft/range.access.begin).  
		  > `ranges::begin(E)` is expression-equivalent to one of the following, in sequential order:
		- [UFCS: Customization and Extension](https://brevzin.github.io/c++/2019/08/22/ufcs-custom-extension/) (2019-08-22) 
			- Customization
		- [What is unified function call syntax anyway?](https://brevzin.github.io/c++/2019/04/13/ufcs-history/)  (2019-04-13) 
	- [言語機能 C++20 コンセプト](https://cpprefjp.github.io/lang/cpp20/concepts.html) (cpprefjp)
	- [リファレンス concepts](https://cpprefjp.github.io/reference/concepts.html) (cpprefjp)
		- コンセプトのモデル  
		  > ... 意味論的な制約 (モデルとなる条件) を全て満足するとき、  
		  > そのような型 T はコンセプト C の モデル (`model`) である。  
- C++0x Concept
	- [コンセプトは滅びぬ！何度でもよみがえるさ！コンセプトの力こそC++erの夢だからだ！](https://spinor.hatenablog.com/entry/20111215/1323951052) (spinorのブログ)
		- `model` という単語は (型がコンセプトを) 満足するという意味の動詞として、  
		  また、コンセプト要件を満足する型のセットを意味する名詞として、使われるそうです。
		- [ConceptGCC](http://www.generic-programming.org/software/ConceptGCC.html)  ( [江添さんinfo(2015)](https://ezoeryou.github.io/blog/article/2015-08-07-gcc-concept.html) )
		- [ConceptClang](http://www.generic-programming.org/software/ConceptClang/)
		- [コンセプト (C++) - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%B3%E3%82%BB%E3%83%97%E3%83%88_%28C%2B%2B%29)
		- [N2081 (2006) - Concepts (Revision 1)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2081.pdf)  - Douglas Gregor, Bjarne Stroustrup
		- [N2941 (2009) - Simplifying the use of concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2906.pdf)  - Bjarne Stroustrup
		- http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2914.pdf コンセプトが記載された最後の規格書ドラフト
- ISO C++ Paper/Information
	- [C++0x Concepts — Historical FAQs](https://isocpp.org/wiki/faq/cpp0x-concepts-history)
	- [N3629 (2013-04-09) - Simplifying C++0x Concepts](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3629.pdf)  - Doug Gregor

# 実装の研究 - CPO
## GCC libstdc++-v3
- 実装
	- https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/bits/iterator_concepts.h#L928-L976
	- https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/bits/ranges_base.h#L97-L131
	- https://github.com/gcc-mirror/gcc/blob/master/libstdc%2B%2B-v3/include/bits/ranges_base.h#L562
	- ( [初版](https://github.com/gcc-mirror/gcc/commit/6d0dff49ca1539e14647c04cc1bb035ef4c2780b#diff-2ccfc1aaac6d2b578a5ed67e33d93ef8ab6aab88e7a8002ba30026e4dbb79b6a) [bits/range_access.h](https://github.com/gcc-mirror/gcc/blob/6d0dff49ca1539e14647c04cc1bb035ef4c2780b/libstdc%2B%2B-v3/include/bits/range_access.h#L384-L430) , 2019-10-30 ) 
- 状況:
	- [C++20 Support in GCC](https://gcc.gnu.org/projects/cxx-status.html#cxx20)  
	- [Implementation Status - Table 1.9. C++ 2020 Library Features](https://gcc.gnu.org/onlinedocs/libstdc++/manual/status.html#status.iso.2020)

| Library Feature | Status |
| --- | --- |
| Ranges and Concepts | 10.1 |

## LLVM libcxx
- 実装: https://github.com/llvm/llvm-project/blob/main/libcxx/include/__ranges/access.h#L39-L100
- 状況: [libc++ Ranges Status](https://libcxx.llvm.org/docs/RangesStatus.html)
	- [github](https://github.com/llvm/llvm-project/tree/main/libcxx/include) : `<concepts>` はあるけど `<ranges>` はまだない。
	- 2021-04-30 に初版がコミットされた。`include/__ranges/access.h`.

| Section | Description | Dependencies | Patch |
| --- | --- | --- | --- |
| \[range.access\] | ranges::begin, end, cbegin, cend, rbegin, rend, crbegin, and crend | \[iterator.concepts\] | [D100255](https://reviews.llvm.org/D100255) |

## MS STL
- 実装: https://github.com/microsoft/STL/blob/main/stl/inc/xutility#L2101-L2168
	- [blame](https://github.com/microsoft/STL/blame/cb3718935a7c506196d48f25a68447657f731744/stl/inc/xutility#L2101-L2168)
	- [初版](https://github.com/microsoft/STL/commit/c5e2e3f799ba1892c2143eb65a8dd1dd5024edb3) (2019-09-10)
```
inc$ grep -nr "inline constexpr" * | grep begin
xutility:2167:        inline constexpr _Begin::_Cpo begin;
  :
inc$ grep -nr "inline constexpr" * | grep distance
xutility:3063:    inline constexpr _Distance_fn distance{_Not_quite_object::_Construct_tag{}};
  :
```
- `std::ranges::distance` : https://github.com/microsoft/STL/blob/main/stl/inc/xutility#L3009-L3063
	-  [\[range.iter.ops\] 2](https://timsong-cpp.github.io/cppwp/n4861/range.iter.ops#2)
	-  [ADLを無効にする関数定義](https://cpprefjp.github.io/article/lib/disable_adl_function.html) (cpprefjp) ※
- `std::ranges::find` :  https://github.com/microsoft/STL/blob/main/stl/inc/xutility#L5352-L5421
	- [\[algorithms.requirements\] 2](https://timsong-cpp.github.io/cppwp/n4861/algorithms.requirements#2)
	- [What is a niebloid?](https://stackoverflow.com/questions/62928396/what-is-a-niebloid) (stackoverflow)
