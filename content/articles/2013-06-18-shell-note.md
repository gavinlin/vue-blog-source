---
title: shell使用笔记
date: 2013-06-18 14:45
category: linux
tags: shell
---

变量赋值

    :::java
    #输入的第一个参数赋值给INTENT
    INTENT ＝ $1
<!-- excerpt -->

无输入和输出的函数

    :::java
    function xxx()
    {

    }

#调用的话
xxx

条件判断

    :::java
    if [ "$TARGET_PRODUCT" = "rk30sdk"  ]; then
        if [ -z "$INTENT" ]; then
            help
        else
            .......
        fi
    else
        echo "please lunch to chose a rockchip product"
    fi

sed使用

    :::java
    #这里使用了嵌套，先找到该行，再把true替换成false
    sed -i '/BUILD_WITH_SUPERUSER/{s/true/false/g}' $ANDROID_BUILD_TOP/device/rockchip/$TARGET_PRODUCT/BoardConfig.mk


case的使用

    :::java
    case $1 in
        "user" )
            prepare_user
            ;;
        "develop" )
            prepare_develop
            ;;
        * )
            echo "input error"
            ;;
    esac
