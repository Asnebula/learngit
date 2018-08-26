### 1.关于注解
```
	def ff(func):
		def ff2(arg):
			arg=int(arg)
			return func(arg)
		return ff2

	@ff #ff3=ff(ff3)
	def ff3(a):
		return a*3
```
	#在ff3的顶部第一行加入@ff相当于执行ff3=ff(ff3)，返回的是一个ff2的函数
	#所以可以ff3(arg)相当于ff(ff3)(arg);这里把ff3的参数a进行了int转换
### 2.关于@wraps
```
	def f1(a):
		return a+1

	@wraps(f1) #@wraps的作用是使被包装的函数（此处是f2）保持与f1相同的属性
	def f2(arg):
		arg=int(arg)
		return f1(arg)
```
	具体来说：如果没有@wraps，那么globals()中是 'f2': <function __main__.f2>,即f2.__name__是f2
	         如果有@wraps，那么globals()中是 'f2': <function __main__.f1>,即f2.__name__是f1，保持与@wraps(f1)中的参数(这里是f1)相同
### 3.更进一步关于wraps的参数
```
	g={'aabfoo':foo,'wraps':wraps,'a5c_processor_arg':_ensure_tuple}
	l={}
	def foo(a):
		return a
	exec_str=dedent(
	'''
	@wraps(aabfoo)
	def foo1(arg):
		#相当于调用a5c_processor_arg对应的(比如_ensure_tuple)函数 # 参数先经过这一步操作
		arg = a5c_processor_arg(aabfoo, 'arg', arg)
		#再调用原函数
		return aabfoo(arg)
	''')
```
	这里需要说明的是执行完exec(exec_str,g,l)后
		l中'foo1': <function __main__.foo>} 而不是'foo1': <function __main__.aabfoo>}！，
		所以wraps最终作用到的是命名空间g中的aabfoo对应的函数而不是aabfoo这个名字 
	（一般情况下命名空间g/l中函数的key和value是同名的，只有通过
		1.赋值(本例)
		2.@wraps(m)(key:被装饰函数名,value:m[@wraps注解的函数参数])两种方式才会发生不同）详见2
		3.普通注解详见4
### 4.普通注解与加入@wraps的注解
```
	def ff(func):
		def ff2(arg):
			arg=int(arg)
			return func(arg)
		return ff2

	@ff # ff3=ff(ff3)
	def ff3(a):
		return a*3
```
	globals中会'ff3': <function __main__.ff.<locals>.ff2>,

```
	from functools import wraps
	def ff(func):
		@wraps(func)
		def ff2(arg):
			arg=int(arg)
			return func(arg)
		return ff2

	@ff # ff3=ff(ff3)
	def ff3(a):
		return a*3
```
	globals()中会'ff3': <function __main__.ff3>修正了ff2的__name__为func(这里为ff3）
### 5.注解中传参
```
	def log(text):
		def decorator(func):
			def wrapper(*args, **kw):
				print('%s %s():' % (text, func.__name__))
				return func(*args, **kw)
			return wrapper
		return decorator
	
	@log('execute') #now = log('execute')(now) #当然如果now本身含有参数可继续(),而且会传到wrapper中的(*arg,**kw)
	def now():
		print('2015-3-25')
	>>>now()	
	execute now():
	2015-3-25
```
