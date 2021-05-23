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

# 参考資料
- 江添さんの解説
	- 2010-09-20 [コンセプトの経緯](https://cpplover.blogspot.com/2010/09/blog-post_8970.html)
	- 2015-05-13 [N4381: Suggested Design for Customization Points](https://cpplover.blogspot.com/2015/05/c2015-04-pre-lenexa-mailings-n4381-n4389.html)
		- `std::swap`, `std::begin`, `std::end` 
- 新しい言語規格ドラフトのレビュー
	- [カスタマイゼーションポイントオブジェクト(CPO)概論](https://onihusube.hatenablog.com/entry/2020/06/26/225920) (地面を見下ろす少年の足蹴にされる私)  
	  > C++における名前探索では修飾名探索と非修飾名探索を行なった後、引数依存名前探索（ADL）を行いオーバーロード候補集合を決定します。この時、非修飾名探索の結果に関数以外のものが含まれているとADLは行われません。逆に言うと、ADLは関数名に対してしか行われません。つまり、関数オブジェクトに対してはADLは発動しません（6.5.2 Argument-dependent name lookup [basic.lookup.argdep]）
		- [C++17のカスタマイゼーションポイントとC++20からのカスタマイゼーションポイントオブジェクトの対応と一覧](https://onihusube.hatenablog.com/entry/2020/06/26/225920#C20%E3%81%AE%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%9E%E3%82%A4%E3%82%BC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%9D%E3%82%A4%E3%83%B3%E3%83%88%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88) 
	- [Customization Point Object](https://yohhoy.hatenadiary.jp/entry/20190403/p1) (yohhoyの日記)
	- [Customization pointについて](http://tk0xleader.blog.shinobi.jp/c--/%E3%80%90c--2a%E3%80%91customization%20point%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6) (tk-xleaderのブログ)
		- `using std::swap;` や、`using namespace std;` として `swap` を非修飾名で呼び出す方法
			- （探索先にstd名前空間を加えてADLを意図的に引き起こす手段） 
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
		- std::tag_invoke (C++23 で導入を目指している)
	- [Niebloids and Customization Point Objects](https://brevzin.github.io/c++/2020/12/19/cpo-niebloid/) (2020-12-19, Barry Revzin's github)
		- [std::ranges::swap (C++20)](https://cpprefjp.github.io/reference/concepts/swap.html) (cpprefjp)
		- [\[customization.point.object\]](http://eel.is/c++draft/customization.point.object#def:customization_point_object) (c++draft)  
		  > In the standard library, customization point object is a specific term of art that is a semiregular, function object that is const-invocable. And the standard library has a whole bunch of them in the std::ranges namespace (begin, end, swap, etc.) but even a few in plain old std (e.g. strong_order).
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
