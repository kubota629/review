# review
[P2279R0: We need a language mechanism for customization points](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2021/p2279r0.html)  - 2021-01-15 Barry Revzin  
　参考資料: [(地面を見下ろす少年の足蹴にされる私)](https://onihusube.hatenablog.com/#P2279R0-We-need-a-language-mechanism-for-customization-points)

Existing Static Polymorphism Strategies
1. Class Template Specification
2. ADL
	1. "pure" ADL
	2. Customization Point Objects (aka CPOs)  
	   [N4381: Suggested Design for Customization Points](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4381.html)  
	   2015-03-11 Eric Niebler
	3. tag_invoke  
	   [P1895: tag_invoke: A general pattern for supportingcustomisable functions](http://open-std.org/JTC1/SC22/WG21/docs/papers/2019/p1895r0.pdf)  
	   2019-10-07 Lewis Baker, Eric Niebler, Kirk Shoop

# Customization Point Objects (CPO)
C++20で導入された新しいデザインパターン。[\[customization.point.object\]](https://timsong-cpp.github.io/cppwp/n4861/customization.point.object)  
  
semiregular な関数オブジェクト。(callable function object)  
「制約のあるADL (constrained ADL dispatch)」を行うために存在する。  
  
C++20 Concept で必要なチェックをした後に、ADL が有効な文脈に実引数を渡してあげると  
ADL を制御下に置くことができるようになる。

## ___customization point___  - [N4381](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4381.html) (2015-03-11)  
C++標準ライブラリの関数で、user’s namespace にある user-defined types によって  
オーバーロードすることができ、かつ、ADL によって見つかるもの。 

C++標準ライブラリには、ユーザー側で挙動を変更できる箇所がすでにいくつか存在していた。
- `std::swap`
- `std::begin`
- `std::end` 

[N4381](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2015/n4381.html) では、
1. customization point を定義する際の、現行アプローチの使い勝手の問題点
2. 将来の customization point 定義において利用できるデザインパターンの提示

を行うこととしている。

## 背景

ADL

- Rangeライブラリ ( C++20: N4861 [PDF](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/n4861.pdf)/[HTML](https://timsong-cpp.github.io/cppwp/n4861/) )
	- \<concepts\> - [18 Concepts library \[concepts\]](https://timsong-cpp.github.io/cppwp/n4861/#concepts)
		- `ranges::swap` [\[concept.swappable\]](https://timsong-cpp.github.io/cppwp/n4861/concept.swappable) 
	- \<ranges\> - [24 Ranges library \[ranges\]](https://timsong-cpp.github.io/cppwp/n4861/#ranges)
		- `ranges::begin` [\[range.access.begin\]](https://timsong-cpp.github.io/cppwp/n4861/range.access.begin)
		- `ranges::end` [\[range.access.end\]](https://timsong-cpp.github.io/cppwp/n4861/range.access.end)

# C++0x Concept Maps (concept_map)


# 参考資料
- 江添さんの解説
	- 2010-09-20 [コンセプトの経緯](https://cpplover.blogspot.com/2010/09/blog-post_8970.html)
	- 2015-05-13 [N4381: Suggested Design for Customization Points](https://cpplover.blogspot.com/2015/05/c2015-04-pre-lenexa-mailings-n4381-n4389.html)
		- C++11 `std::swap`, `std::begin`, `std::end` 
- 新しい言語規格ドラフトのレビュー・解説
	- [What are customization point objects and how to use them?](https://stackoverflow.com/questions/53495848/what-are-customization-point-objects-and-how-to-use-them)  
	- 地面を見下ろす少年の足蹴にされる私
		- [カスタマイゼーションポイントオブジェクト (CPO) 概論](https://onihusube.hatenablog.com/entry/2020/06/26/225920) (2020-06-26)
		  > C++における名前探索では修飾名探索と非修飾名探索を行なった後、引数依存名前探索（ADL）を行いオーバーロード候補集合を決定します。この時、非修飾名探索の結果に関数以外のものが含まれているとADLは行われません。逆に言うと、ADLは関数名に対してしか行われません。つまり、関数オブジェクトに対してはADLは発動しません（[6.5.2 Argument-dependent name lookup \[basic.lookup.argdep\]](https://timsong-cpp.github.io/cppwp/n4861/basic.lookup.argdep#3.3)）
			- これまでのカスタマイゼーションポイント
				- 従来の CP関数 を CPO に置き換えたかったが、互換性の問題からできなかったようです
				- そのため、CPO は別の名前空間に同名で定義されています。
			- C++20のカスタマイゼーションポイントオブジェクト
				- [C++17 CPO, C++20 CPO の対応と一覧](https://onihusube.hatenablog.com/entry/2020/06/26/225920#C20%E3%81%AE%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%8i2%A4%E3%82%BC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%9D%E3%82%A4%E3%83%B3%E3%83%88%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88)
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
	- [Customization pointについて](http://tk0xleader.blog.shinobi.jp/c--/%E3%80%90c--2a%E3%80%91customization%20point%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6) (tk-xleaderのブログ)
		- `using std::swap;` や、`using namespace std;` として `swap` を非修飾名で呼び出す方法
			- (探索先にstd名前空間を加えてADLを意図的に引き起こす手段)
(stackoverflow)
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
