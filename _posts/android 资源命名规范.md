---
layout:     post
title:      android 资源命名规范
date:       2019-12-20
author:     guuguo
catalog: true
tags:
    - Android
    - 规范

---

这样命名的优点：

1. 资源文件按屏幕排序。<where> 指明了资源文件所属的屏幕，所以你能很快找到某个特定界面的资源文件。
2. 更好的组织结构。Android Studio 中的文件是按照字母顺序的，因此 WHAT 和 WHERE 前缀就起到了文件分类的作用。
3. 更高效的自动完成。通常只需输入 WHAT 或 WHERE 就足以将自动完成提供的选项缩小到合适的程度。
4. 合乎逻辑的资源文件命名，会让你的 Android 项目结构更加清晰。



> where 通常是模块名,或者未分模块但明确功能划分的功能名

## drawable

> `<where>_<description>_<size>`

drawable 如 `coach_infoicon_large`

> `common_<description>_<size>`

drawable 可能会被复用的放在common 模块 以common为前缀 如 `common_infoicon_large`



## layout

>  `<where>_<what>_<description>`

示例:

- shop_fragment_order_evaluate
- shop_item_image_add

<WHAT> 是下面的其中一个：

| 前缀 | 用法 |
| ---- | ---- |
| Ac   |      |
|      |      |
|      |      |
|      |      |



## DIMENSION

> ` <what>_<description>`

应当只定义有限的 dimension，并尽量复用它们。

示例:

- height_toolbar
- keyline_listtext
- textsize_medium
- size_menu_icon
- height_menu_profileimage

<what> 前缀可以用下面列举的这些：

| 前缀      | 用法                                           |
| --------- | ---------------------------------------------- |
| width     | width in dp                                    |
| height    | height in dp                                   |
| size      | if width==height                               |
| margin    | margin in dp                                   |
| padding   | padding in dp                                  |
| elevation | elevation in dp                                |
| keyline   | absolute keyline measured from view edge in dp |
| textsize  | size of text in sp                             |



## id

> ` <what>_<description>`

id 命名 开头以何种view 命名。如` tv_time_desc`

what 遵循 驼峰首字母缩写 案例如下:

| View             | Prefix |
| ---------------- | ------ |
| ImageView        | iv     |
| TextView         | tv     |
| Group            | Group  |
| LineaLayout      | ll     |
| ConstraintLayout | cl     |
| FramgLayout      | fl     |

