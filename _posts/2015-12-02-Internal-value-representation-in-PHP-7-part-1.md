---
title: PHP7数据的内部实现 —— 上
tags: 技术 PHP
--- 

摘要：翻译自 http://nikic.github.io/2015/05/05/Internal-value-representation-in-PHP-7-part-1.html 介绍了PHP7的内部实现。

上一篇文章讲了PHP7中对hashtable的实现的改进。这篇将探索PHP Value的新的描述方式。

因为篇幅的原因，分成两部分：上篇来描述 zval (Zend value) 的实现在PHP 5 和 PHP 7 的不同，包括对引用（references）的实现。下篇将详述不同数据类型的细节，比如字符串、对象等。

## PHP5中的Zvals 
在PHP5中，zval的数据结构是这样定义的：

```c
typedef struct _zval_struct {
    zvalue_value value;
    zend_uint refcount__gc;
    zend_uchar type;
    zend_uchar is_ref__gc;
} zval;
```

一个zval包含了一个值（value），一个类型（type），以及一些其它的 __gc 附加信息。value是一个联合体（union），以便存储各种不同类型的数据：

```c
typedef union _zvalue_value {
    long lval;                 // 存储布尔值，整数，以及资源
    double dval;               // 存储浮点数
    struct {                   // 存储字符串
        char *val;
        int len;
    } str;
    HashTable *ht;             // 存储数组
    zend_object_value obj;     // 存储对象
    zend_ast *ast;             // 存储常量表达式
} zvalue_value;
```
C语言中的联合体（union）每次只能保存一个成员数据，联合体的长度等于各成员中最长的长度。所有的成员数据被存储在同一个内存地址，在访问特定成员时，数据类型会被转换。比如你读取联合体的lval，联合体的值将会被解释成一个有符号的整数。如果你读取dval成员，联合体的值就会被当作double类型。依此类推。

为了正确的读取联合体成员，zval中使用type来标记联合体中正在被使用的成员类型，type是一组定义好的整数：

```c
#define IS_NULL     0      /* Doesn't use value */
#define IS_LONG     1      /* Uses lval */
#define IS_DOUBLE   2      /* Uses dval */
#define IS_BOOL     3      /* Uses lval with values 0 and 1 */
#define IS_ARRAY    4      /* Uses ht */
#define IS_OBJECT   5      /* Uses obj */
#define IS_STRING   6      /* Uses str */
#define IS_RESOURCE 7      /* Uses lval, which is the resource ID */

/* Special types used for late-binding of constants */
#define IS_CONSTANT 8
#define IS_CONSTANT_AST 9
```

## PHP5中的引用计数
PHP5的zval存在堆中，PHP需要跟踪这些zval是否正在被使用，或者应该被释放掉。为了达到这个目的，使用了引用计数：refcount__gc 记录了一个zval当前被“引用”的次数。例如：$a = $b = 42 ，42这个值被两个变量引用，所以她的refcount是2。当refcount到0的时候，表示这个值没有用了，可以被释放掉了。

注意这里提到的“引用”（值被使用的次数）和PHP中的引用（&）是两回事。文章将使用“引用”和“PHP引用”来区别这两个概念。现在提到的都是“引用”，所以请先忽略“PHP引用”

另外一个和引用相关的概念叫做“写时复制（copy on write）”：一个zval如果没有被修改的话，可以被多个使用者共享使用。需要改变这个值时，才需要复制（分离）一次，然后修改这个复制出来的值。

看一个关于写时复制和zval销毁的例子：

```c
$a = 42;   // $a         -> zval_1(type=IS_LONG, value=42, refcount=1)
$b = $a;   // $a, $b     -> zval_1(type=IS_LONG, value=42, refcount=2)
$c = $b;   // $a, $b, $c -> zval_1(type=IS_LONG, value=42, refcount=3)

// The following line causes a zval separation
$a += 1;   // $b, $c -> zval_1(type=IS_LONG, value=42, refcount=2)
           // $a     -> zval_2(type=IS_LONG, value=43, refcount=1)

unset($b); // $c -> zval_1(type=IS_LONG, value=42, refcount=1)
           // $a -> zval_2(type=IS_LONG, value=43, refcount=1)

unset($c); // zval_1 is destroyed, because refcount=0
           // $a -> zval_2(type=IS_LONG, value=43, refcount=1)
```

引用计数有一个致命的缺陷：循环引用。为了解决循环引用，PHP加入了一个循环收集器（cycle collector）。zval的引用计数递减，并且可能成为环上一个部分的时候，zval会被写入到“root buffer”。一旦root buffer满了的时候，可能的环会被标记并被GC收集清理。

为了支持这个额外的环收集器， 实际使用的zval结构，是这样的：

```c
typedef struct _zval_gc_info {
    zval z;
    union {
        gc_root_buffer       *buffered;
        struct _zval_gc_info *next;
    } u;
} zval_gc_info;
```

zval_gc_info 结构体嵌入了一个zval，还有一个附加的指针——u，联合体，所以u也是一个可以有两个指针类型的指针。buffered指针用来存储在root buffer中zval的位置，这样可以在环清理之前，把已经销毁的zval从环中移除。next用于环清理的时候。

## 改变的动机
先来看一下大小（64位系统下）：zvalue_value有16字节大，因为str和obj需要这么大空间。整个zval结构有24个字节大小（包括对齐），zval_gc_info 有32字节。而且，这些堆上分配时，还需要额外的16字节。这样我们需要为每个zval分配48个字节的空间。

这样我们开始思考zval的实现是否效率低下。想一下简单的存储一个整形，数据需要8字节。类型标记无论如何都要需要，他只有一个字节，但是需要8个字节对齐。

这16个字节是我们“需要”的，我们还需要另外16个字节处理引用计数和循环引用收集，还有另外16字节分配开销。这还不算我们真正在内存分配和释放时操作的开销，这些开销也很大。

问题来了：简单的整数，是否真多需要存储引用计数、循环引用收集、堆分配值？当然不用！

这里罗列了一些PHP5中zval实现上的问题：
+ zval总是在堆上分配空间
+ zval总是引用计数，总是保留处理循环引用的收集器的信息，即便没什么意义。
+ 直接引用计数zvals，在object和resource上，实际上是重复计数了。（背后的原因，下一篇再说）
+ 一些情况下，会产生大量的间接寻址。比如访问一个在变量中的对象，会需要四层指针引用。下一章会详细说明。
+ 直接zval引用计数，也已意味着这些值只能在zval之间共享。例如：一个字符串就不能既用于zval又用于hashtable的键值。（除非，hashtable的键值也用zval）

## PHP7中的zval
来看PHP7中的zval实现。最根本的改变，是zval不再独自在堆上分配内存，而且不在zval结构中存储引用计数了。 那些指针指向的复杂的值（像字符串，数组或者对象）他们自身就可以保存引用计数。（Instead any complex values they may point to (like strings, arrays or objects) will store the refcount themselves. ？）这有以下优点：

＋ 简单类型的值，不再需要请求分配内存，不使用引用计数。
＋ 不会再有重复的引用计数，在object中，只有object内部的引用计数。
＋ 因为现在的引用计数保存在值自身中，所以这些值可以被不同zval结构体分别共享。一个字符串，即可用在zval中，也可以用在hashtable的key中。
＋ 少了很多间接寻址。意味着，在你需要寻找值时使用的指针变少了。

现在新的zval是这样定义的：
```c
struct _zval_struct {
    zend_value value;
    union {
        struct {
            ZEND_ENDIAN_LOHI_4(
                zend_uchar type,
                zend_uchar type_flags,
                zend_uchar const_flags,
                zend_uchar reserved)
        } v;
        uint32_t type_info;
    } u1;
    union {
        uint32_t var_flags;
        uint32_t next;                 // hash collision chain
        uint32_t cache_slot;           // literal cache slot
        uint32_t lineno;               // line number (for ast nodes)
        uint32_t num_args;             // arguments number for EX(This)
        uint32_t fe_pos;               // foreach position
        uint32_t fe_iter_idx;          // foreach iterator index
    } u2;
};
```
第一个成员和PHP5中的基本一致，zend_value也是一个value联合体。第二个成员是一个整形村粗的类型信息，使一个联合体进一步分为单独的字节（你可以先忽略ZEND_ENDIAN_LOHI_4宏定义，这个是用来真对不同平台的处理字节序的）。成员结构中最重要的两个子成员：type 和 type_flags，type和以前的差不多，type_flags我稍后会说到。

这里存在小问题：value成员是8字节大小，由于结构体会被填充，所以即便只是再增加一个字节，也会扩大到16个字节。我们明显不需要8个字节存储type，这就是为什么会有另外一个联合体u2，默认情况下是不使用的，但是可以设定为存储4个字节的数据。不同的成员对应这个数据槽不同的用法。

value联合体，在PHP7中，看起来有一些小不同：
```c
typedef union _zend_value {
    zend_long         lval;
    double            dval;
    zend_refcounted  *counted;
    zend_string      *str;
    zend_array       *arr;
    zend_object      *obj;
    zend_resource    *res;
    zend_reference   *ref;
    zend_ast_ref     *ast;

    // Ignore these for now, they are special
    zval             *zv;
    void             *ptr;
    zend_class_entry *ce;
    zend_function    *func;
    struct {
        ZEND_ENDIAN_LOHI(
            uint32_t w1,
            uint32_t w2)
    } ww;
} zend_value;
```

首先，记住现在的zval是8字节的，而非16字节的。他只能直接存储整型（lval）和双精度浮点数（dval），其它所有的一切，都是指针。所有的指针类型（标记special那行注释的上面）使用引用计数，相同的头定义在zend_refconted 中:
```c
struct _zend_refcounted {
    uint32_t refcount;
    union {
        struct {
            ZEND_ENDIAN_LOHI_3(
                zend_uchar    type,
                zend_uchar    flags,
                uint16_t      gc_info)
        } v;
        uint32_t type_info;
    } u;
};
```
这个结构体中包含引用计数refcount。另外还包括类型type，一些标记flags，和gc信息gc_info。type是zval的type的复制来的，允许GC不用保存zval就可以区别不同的引用计数结构体类型。flags有不同的使用目的，下一章会按照类型分别介绍。

gc_info 相当于旧zval中的buffered。但是没有存储指针到root buffer中，而是包含了一个它的索引。因为root buffer已经扩充大小（10000元素）足够使用16位整数，替代一个64位指针。gc_info还编码了节点的“颜色”，用来标记回收中的节点。

Zval内存管理
前面已经提到了，zval不在单独的使用堆空间。但是显然需要存储空间，他说怎么办到的？当zval是堆上分配存储的结构体的一部分的时候，他们直接嵌入结构体中。例如，一个hashtable桶将直接把zval加入到自己的结构中，而不是保存一个指针，再指向一个独立的zval。函数的编译的变量表或对象的属性表，将分配一个区块，而不是存储指针指向单独的zval。现在zval存储通常会减少一级间接寻址。以前是 zval * 现在是 zval。

当zval在新的地方被使用的时候，之前是复制zval* ，增加它的引用计数。现在，复制zval的内容，可能增加值指向的引用计数，如果值使用了引用计数（？）

但是PHP如何知道一个value是不是使用了引用计数？他不能仅仅通过type来判断，因为一些类型比如字符串和数字，并不是总是使用引用计数的。zval中的type_info里面有一个bit，用来标记zval是不是引用计数。这里有其它的按位编码的类型属性：

```c
#define IS_TYPE_CONSTANT            (1<<0)   /* special */
#define IS_TYPE_IMMUTABLE           (1<<1)   /* special */
#define IS_TYPE_REFCOUNTED          (1<<2)
#define IS_TYPE_COLLECTABLE         (1<<3)
#define IS_TYPE_COPYABLE            (1<<4)
#define IS_TYPE_SYMBOLTABLE         (1<<5)   /* special */
```

三个重要的属性， “refcounted（引用计数）”, “collectable（可收集）” and “copyable（可复制）”。我们已经说过引用计数的意义。collectable

## 类型

## 引用

## 上层封装