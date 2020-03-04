---
title: "lambda表达式"
date: 2020-02-16T13:45:00+08:00
draft: false
tags: 
 - C++ 11
categories: 
 - C/C++
---











 <!--more-->

A lambda expression is a mechanism for specifying a function object. The primary use for a lambda is to specify a simple action to be performed by some function. For example: 

```
	vector<int> v = {50, -10, 20, -30};

	std::sort(v.begin(), v.end());	// the default sort
	// now v should be { -30, -10, 20, 50 }

	// sort by absolute value:
	std::sort(v.begin(), v.end(), [](int a, int b) { return abs(a)<abs(b); });
	// now v should be { -10, 20, -30, 50 }
```

 The argument  `[](int a, int b) { return abs(a)<abs(b); } ` is a "lambda" (or "lambda function" or "lambda expression"), which specifies an operation that given two integer arguments a and b returns the result of comparing their absolute values. 

 A lambda expression can access local variables in the scope in which it is used. For example: 

```
	void f(vector<Record>& v)
	{
		vector<int> indices(v.size());
		int count = 0;
		generate(indices.begin(),indices.end(),[&count](){ return count++; });

		// sort indices in the order determined by the name field of the records:
		std::sort(indices.begin(), indices.end(), [&](int a, int b) { return v[a].name<v[b].name; });
		// ...
	}
```

 Some consider this "really neat!"; others see it as a way to write dangerously obscure code. IMO, both are right. 

 The **[&]** is a "capture list" specifying that local names used will be passed by reference.  We could have said that we wanted to "capture" only **v**, we could have said so: **[&v]**. Had we wanted to pass **v** by value, we could have said so: **[=v]**. Capture nothing is **[]**, capture all by references is **[&]**,  and capture all by value is **[=]**. 

 If an action is neither common nor simple, I recommend using a named function object or function. For example, the example above could have been written: 

```
	void f(vector<Record>& v)
	{
		vector<int> indices(v.size());
		int count = 0;
		generate(indices.begin(),indices.end(),[&](){ return ++count; });

		struct Cmp_names {
			const vector<Record>& vr;
			Cmp_names(const vector<Record>& r) :vr(r) { }
			bool operator()(int a, int b) const { return vr[a].name<vr[b].name; }
		};

		// sort indices in the order determined by the name field of the records:
		std::sort(indices.begin(), indices.end(), Cmp_names(v));
		// ...
	}
```

 For a tiny function, such as this Record name field comparison, the function object notation is verbose, though the generated code is likely to be identical. In C++98, such function objects had to be non-local to be used as template argument; in C++11 this is no longer necessary. 

 To specify a lambda you must provide 

- its capture list: the list of variables it can use (in addition to its arguments), if any (**[&]** meaning "all local variables passed by reference" in the Record comparison example). If no names needs to be captured, a lambda starts with plain **[]**. 
- (optionally) its arguments and their types (e.g, **(int a, int b)**) 
- The action to be performed as a block (e.g., **{ return v[a].name<v[b].name; }**). 
- (optionally) the return type using the [new suffix return type syntax](http://www.stroustrup.com/C++11FAQ.html#suffix-return); but typically we just deduce the return type from the return statement. If no value is returned **void** is deduced. 

  See also: 

-  Standard 5.1.2 Lambda expressions 
- [N1968=06-0038] Jeremiah Willcock, Jaakko Jarvi, Doug Gregor, Bjarne Stroustrup, and Andrew Lumsdaine:  [Lambda expressions and closures for C++](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n1968.htm) (original proposal with a different syntax) 
- [N2550=08-0060] Jaakko Jarvi, John Freeman, and Lawrence Crowl: [Lambda Expressions and Closures: Wording for Monomorphic Lambdas (Revision 4)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2550.pdf) (final proposal). 
- [N2859=09-0049] Daveed Vandevoorde: [New wording for C++0x Lambdas](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2859.pdf). 