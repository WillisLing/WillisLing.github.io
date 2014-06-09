---
layout: post
title: What is stack unwinding?
categories:
- Programming
tags:
- c++

---

Stack unwinding is usually talked about in connection with exception handling. Here's an example:

{% highlight cpp %}
void func( int x )
{
    char* pleak = new char[1024]; // might be lost => memory leak
    std::string s( "hello world" ); // will be properly destructed

    if ( x ) throw std::runtime_error( "boom" );

    delete [] pleak; // will only get here if x != 0
}

int main()
{
    try
    {
        func( 10 );
    }
    catch ( const std::exception& e )
    {
        return 1;
    }

    return 0;
}
{% endhighlight %}

Here memory allocated for `pleak` will be lost if exception is thrown, while memory allocated to s will be properly released by `std::string` destructor in any case. The objects allocated on the stack are "unwound" when the scope is exited (here the scope is of the function `func`.) This is done by the compiler inserting calls to destructors of automatic (stack) variables.

Now this is a very powerful concept leading to the technique called [RAII](http://en.wikipedia.org/wiki/RAII), that is ***Resource Acquisition Is Initialization***, that helps us manage resources like memory, database connections, open file descriptors, etc. in C++.

[MSVC 资料](http://msdn.microsoft.com/en-us/library/hh254939.aspx)