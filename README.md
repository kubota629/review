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
- 新しい言語規格ドラフトのレビュー
	- [カスタマイゼーションポイントオブジェクト(CPO)概論](https://onihusube.hatenablog.com/entry/2020/06/26/225920) (地面を見下ろす少年の足蹴にされる私)  
	  > C++における名前探索では修飾名探索と非修飾名探索を行なった後、引数依存名前探索（ADL）を行いオーバーロード候補集合を決定します。この時、非修飾名探索の結果に関数以外のものが含まれているとADLは行われません。逆に言うと、ADLは関数名に対してしか行われません。つまり、関数オブジェクトに対してはADLは発動しません（6.5.2 Argument-dependent name lookup [basic.lookup.argdep]）
	- [Customization Point Object](https://yohhoy.hatenadiary.jp/entry/20190403/p1) (yohhoyの日記)
	- [Customization pointについて](http://tk0xleader.blog.shinobi.jp/c--/%E3%80%90c--2a%E3%80%91customization%20point%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6) (tk-xleaderのブログ)
		- `using std::swap;` や、`using namespace std;` として `swap` を非修飾名で呼び出す方法
			- （探索先にstd名前空間を加えてADLを意図的に引き起こす手段） 
- C++言語規格の資料
	- [C++の名前解決](https://prettysoft.hatenablog.com/entry/20101128/1497356882) (プログラミングの教科書を置いておくところ)
	- [C++の名前解決 ADL その2](https://prettysoft.hatenablog.com/entry/20101129/204558) (プログラミングの教科書を置いておくところ)
		- ADL と関数テンプレート : N1905, N4713
		- ADL と関数オブジェクト : N3000
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
		- ( Nxxxx は Working Draft の Update であった。 - 参考: [cpprefjp - C++国際標準規格](https://cpprefjp.github.io/international-standard.html) )
	- [std::ranges::swap (C++20)](https://cpprefjp.github.io/reference/concepts/swap.html) (cpprefjp)
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
