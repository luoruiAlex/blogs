##### 读取文件方法
- `read()`一次读取文件所有内容，`read(size)`指定一次读取的大小，`readline()`一次读一行

##### io.StringIO、io.BytesIO
- 在内存中
- 和文件IO有一致的接口，比如`write()` `readline()` 也有自己的接口getvalue()

##### 操作文件、目录
- `os.environ.get('PATH')`
- `os.path.abspath('.')`
- `os.mkdir('/Users/michael/testdir')` `os.rmdir('/Users/michael/testdir')` `os.mkdirs` `os.listdir()` 
- `os.path.isdir()` `os.path.isfile()`
- `os.getcwd()` `os.path.abspath()` `os.path.dirname()`
- `os.path.join()` `os.path.split()` `os.path.splitext()` `os.path.exists()`
- `os.rename('test.txt', 'test.py')` `os.remove('test.py')`
- `shutil.copy(src, dst)` `shutil.copytree(src, dst)` `shutil.move(src, dst)
- `os.chdir()`
- `os.walk()`

#### 序列化
- `json.dumps(obj, default=lambda obj: obj.__dict__)` 如果obj不是dict，可指定obj to dict函数
- `json.loads(json_str, object_hook=dict2object)`     如果obj不是dict，可指定dict to obj函数
