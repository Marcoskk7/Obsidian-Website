# GO基础

module是一个公司, package是一个小部分, 一个 go.mod 会声明一个 module, 而一个 module 下会包含很多 package, ./... 表示递归当前 module 下的所有 package, 如果下面有子 module, 那么不会递归子 module 下的内容
go.mod 中的 indirect 模块, 表示的是项目直接依赖的依赖, 因此叫间接依赖