---
layout:     post
title:      "block引入外部变量"
subtitle:   ""
date:       2016-10-10 15:28:00
author:     "Ryan"
header-img: "img/home-bg-o.jpg"
tags:
    - Tips
    - iOS开发
---

	int main() {
    int i = 2;
    NSNumber *num = @3;
    long (^myBlock)(void) = ^long() {
        return i * num.intValue;
    };
    myBlock();
    return 0;
	}
	
##### block的结构体

	struct __block_impl {
     /*  isa指针,非GC模式下有三种类型:
     *  _NSConcreteStackBlock,保存在栈中的 block，当函数返回时会被销毁
     *  _NSConcreteGlobalBlock,全局的静态 block，不会访问任何外部变量
     *  _NSConcreteMallocBlock,保存在堆中的 block，当引用计数为 0 时会被销毁.
     *  当一个 block 被 copy 的时候，才会将这个 block 复制到堆
     */
  	void *isa;
  	//block 的负载信息（引用计数和类型信息）
  	int Flags;
   	//保留变量
  	int Reserved;
   	//指向 block 函数地址的指针
  	void *FuncPtr;
	};

#####  block的数据结构

	struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    //block中引用的变量在申明block时,被复制到__main_block_impl_0结构体中
    int i;
    NSNumber *num;
    __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _i, NSNumber *_num, int flags=0) : i(_i), num(_num) {
        impl.isa = &_NSConcreteStackBlock;
        impl.Flags = flags;
        impl.FuncPtr = fp;
        Desc = desc;
    }
	};

##### block中的方法

	static long __main_block_func_0(struct __main_block_impl_0 *__cself) {
    int i = __cself->i; // bound by copy
    NSNumber *num = __cself->num; // bound by copy
    return i * ((int (*)(id, SEL))(void *)objc_msgSend)((id)num, 	sel_registerName("intValue"));
	}

	static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {_Block_object_assign((void*)&dst->num, (void*)src->num, 3/*BLOCK_FIELD_IS_OBJECT*/);}

	static void __main_block_dispose_0(struct __main_block_impl_0*src) {_Block_object_dispose((void*)src->num, 3/*BLOCK_FIELD_IS_OBJECT*/);}

##### block数据结构的描述
	static struct __main_block_desc_0 {
    size_t reserved;
    size_t Block_size;
    void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
    void (*dispose)(struct __main_block_impl_0*);
	} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0), __main_block_copy_0, __main_block_dispose_0};

&emsp;

	int main() {
    int i = 2;
    NSNumber *num = ((NSNumber *(*)(Class, SEL, int))(void *)objc_msgSend)(objc_getClass("NSNumber"), sel_registerName("numberWithInt:"), 3);
    //block声明
    long (*myBlock)(void) = ((long (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, i, num, 570425344));
    //block调用
    ((long (*)(__block_impl *))((__block_impl *)myBlock)->FuncPtr)((__block_impl *)myBlock);
    return 0;
	}