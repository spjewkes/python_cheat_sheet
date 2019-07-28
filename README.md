# Python Cheat Sheet
This repo is for general Python information that could be useful for any development work.

## Notes	from 'Transforming Code	into Beautiful,	Idiomatic Python'

This section contains notes from the video by Raymod Hettinger about taking advantage of some of Python's language features. The video can be found [here](https://www.youtube.com/watch?v=OSGv2VnC0go).

### General looping

Use ```xrange``` with Python 2.7 (Python 3 use ```range```). This saves memory by creating an iterator.

Do not use indices to reference items in a list. Do the following instead:

```python
colors = ['blue', 'red', 'green']
for color in colors:
    print color
```

To go backwards through a list use the function ```reversed```.

If you need to index at the same time then use ```enumerate```.

If you want to loop over two lists at the same time then use ```izip``` as follows:

```python
colors = ['blue', 'red', 'green']
names = ['alice', 'bob', 'eve']
for name, color in izip(names, colors):
    print name, color
```

Note that ```zip``` does the same thing but will generate the entire structure in memory beforehand. The benefit of things like ```izip``` is that they may end up reusing the same allocations so there could be performance savings too.

Use ```sorted``` to reorder a list. Try not to use custom comparison functions for sorting as this function may get called _n Log n_  times. Use ```sorted```'s ```key``` functions instead. This will be called just once per item. Note that comparison functions are no longer in Python 3.

### Breaking out of loops

With this general type of code:

```python
blocks = []
while True:
    block = f.read(32)
    if block == '':
        break
    blocks.append(block)
```

Can use ```iter``` with its ```sentinel``` paremeter instead to make the code clearer:

```python
blocks = []
for block in iter(partial(f.read, 32), ''):
    blocks.append(block)
```

With the use of the sentinel value in the secons parameter of ```iter``` the first paramter now expects a callable object. This can be done with the use of ```partial``` to create a function that takes no parameters but returns the result of the read.

Avoiding the use of a boolean to find something in a list by using ```else``` with ```for```. For example with:

```python
def find(seq, target):
    found = False
    for i, value in enumerate(seq):
        if value == target:
            found = True
            break
    if not found:
        return -1
    return i
```

The boolean can be removed as follows:

```python
def find(seq, target):
    for i, value in enumerate(seq):
        if value == target:
            break
    else:
        return -1
    return i
```

The ```else``` is called at the end of the ```for``` loop if there was no ```break```. The only issue with this style is that the ```else``` keyword in this sort of loop is not widely known and may cause some confusion with developers maintaining it. This style was defined by Donald Knuth.

### General dictionary uses

Looping over dictionary keys is simply:

```python
d = { '1': 'alice', '2': 'bob', '3': 'eve' }
for k in d:
    print k
```

Do not use this if you wish to modify the dictionary whilst iterating. You should use ```keys``` which makes a copy, allowing you to make modifications:

```python
for k in d.keys():
    if k == '2':
        del d[k]
```

Looping over keys and values at the same time then use ```iteritems```. Unlike ```items``` this will not create a huge list in memory.

The function ```izip``` can be used to construct dictionaries from lists:

```python
ids = ['1', '2', '3']
names = ['alice', 'bob', 'eve']
d = dict(izip(ids, names))
```

Using ```izip``` over ```zip``` should consume less memory as it will be reused when creating the tuple for each pair of list items.

Use ```defaultdict``` for doing something like maintaining a count of items in a list. For example:

```python
names = [`alice', 'bob', 'eve', 'alice', 'bob', 'alice']
d = defaultdict(int)
for name in names:
    d[name] += 1
```

The ```int``` default behaviour will be to return a 0 with no parameters so the value by default for a dictionary item is zero. We can then increment this by 1 every time. Without ```defaultdict``` the dictionary function ```get``` could be used with its ```default``` parameter.

To create a dictionary where each key groups items from a list (in the below example by string length) you can use ```defaultdict```. For example:

```python
names = ['alice', 'bob', 'eve', 'steve']
d = defaultdict(list)
for name in names:
    key = len(name)
    d[key].append(name)
```

In the above example the ```defaultdict``` will be an empty list is returned if the key does not match an existing list in the dictionary. An older way of doing this would be to use the dicitonary function ```setdefault``` when querying the key.

Sometimes you want to link dictionaries together such that if the key is specified in a particular dicitonary then it may take priority over an entry in another dictionary. A common use of this might be program arguments which override some values in environment variables which in turn may override default values. Use of ```ChainMap``` is the most efficient way to do this.

### Improving clarity

Use keyword arguments for more obscure function parameters to make the code much more readable. It does add a slight performance hit so should be avoided for some cases (e.g. function calls with loops).

When returning tuples from functions. consider using named tuples to make the meaning of the values clearer (i.e. use ```namedtuple```). Named tuples are a sub-class of tuples so they still behave like a tuple.

For clarity and speed, always unpack sequences as follows:

```python
employee = 'alice', 'work', 64
name, place, id = employee
```

Packing should be used to avoid temporary variables. For example swapping:

```python
x = 10
y = 20
tmp = x
x = y
y = tmp
```

Instead just do:

```python
x = 10
y = 20
x, y = y, x
```

This updates the state all at once. It avoid potential mistakes with temporary variables and is much clearer. This work even better if you're making calculations with the old values of state and want to update a set of new variables with the new state in one go.

### Efficieny

Never add strings together. Always use ```join```.

When updating sequences with the following:

```python
del seq[0]
seq.pop(0)
seq.insert(0, 'value')
```

It usually means that the wrong data structure is being used and you should consider replacing with a ```deque``` and use the following instead:

```python
del seq[0]
seq.popleft()
seq.appendleft('value')
```

### Decorators and context managers

Decorators can be used to separate business logic from administration code. For example, if you want to cache a look-up value in functions this could use the ```@cache``` decorator. This is in Python 3 but can easily be written for earlier versions.

Rather than copying a thread's context by calling ```getcontext().copy()``` for some reason use ```with`` instead. For example if you need to change the decimal precision:

```python
with localcontext(Context(prec=50)):
    print Decimal(355) / Decimal(113)
```

So the ```with``` is managing the creation of a new context and cleaning it up. This is far easier to read and maintain. Another use is with opening files as ```with``` will ensure that the file is closed automatically.

Locks is another good example. Looking at the bad way of doing it:

```python
lock = threading.Lock()
lock.acquire()
try:
    print "Critical section 1"
    print "Critical section 2"
finally:
    lock.release()
```

This can be improved as follows:

```python
lock = threading.Lock()
with lock:
    print "Critical section 1"
    print "Critical section 2"
```

A more advanced use of a context manager is an example of trying to remove a file that might not be there:

```python
with ignored(OSError):
    os.remove("file.tmp")
```

Note this is in Python 3.4 but it is possible to write your own version of ```ignored``` that has a ```try``` block which will ```yield``` unless it is an exception to ignore. In which case it will drop to the ```except``` and ```pass```.
