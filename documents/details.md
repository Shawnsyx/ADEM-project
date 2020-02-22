&emsp;这是记录自己学习当中的杂碎。

# train.py 中的小细节
## argparse模块使用。
&emsp;argparse是python用于解析命令行参数和选项的标准模块，类似于linux中的ls指令，后面可以跟着不同的参数选项以实现不同的功能，argparse就可以解析命令行然后执行相应的操作。
使用时需要三步(步骤总结来自于[python中argparse模块使用](https://www.cnblogs.com/iMX8mm/p/10821468.html))：
1. 创建 ArgumentParser() 对象
2. 调用 add_argument() 方法添加参数
3. 使用 parse_args() 解析添加的参数


```python
def parse_args():
	parser = argparse.ArgumentParser()  "创建对象"
	parser.add_argument("--prototype", type=str, help="Prototype to use (must be specified)", default='prototype_state')  "调用方法添加参数"
	args = parser.parse_args()
	return args
```

## os.path模块
&emsp;`os.path.exists()`判断括号里的文件是否存在的意思，括号内的可以是文件路径。而且返回值是Boolean类型。
```python
def create_experiment(config):
	if not os.path.exists(config['exp_folder']):
		os.makedirs(config['exp_folder'])
```

## eval函数
&emsp;`eval()`返回括号里表达式的值