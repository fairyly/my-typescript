# my-typescript

typescript

## tsc -b命令行

- [编译选项](https://zhongsp.gitbooks.io/typescript-handbook/content/doc/handbook/Compiler%20Options.html)

你可以指令任意数量的配置文件：

```
 > tsc -b                                # Run the tsconfig.json in the current directory
 > tsc -b src                            # Run src/tsconfig.json
 > tsc -b foo/prd.tsconfig.json bar  # Run foo/prd.tsconfig.json and bar/tsconfig.json
```

不需要担心命令行上指定的文件顺序 - tsc会根据需要重新进行排序，被依赖的项会优先构建。

tsc -b还支持其它一些选项：

```
--verbose：打印详细的日志（可以与其它标记一起使用）
--dry: 显示将要执行的操作但是并不真正进行这些操作
--clean: 删除指定工程的输出（可以与--dry一起使用）
--force: 把所有工程当作非最新版本对待
--watch: 观察模式（可以与--verbose一起使用）
```

- `package.json`

```
"tsc": "tsc", //执行一次TypeScript编译

"dev": "tsc -w" // 以监控模式运行TypeScript编译器。后台始终保持进程。一旦TypeScript文件变化即会重编译
```


## 参考
- [awesome-typescript](https://github.com/dzharii/awesome-typescript)
- [github:Microsoft/TypeScript]( https://github.com/Microsoft/TypeScript)
- [TypeScript 使用手册: TypeScript]( https://github.com/zhongsp/TypeScript)
- [gitbook: typescript-handbook](https://zhongsp.gitbooks.io/typescript-handbook/content/)
- [在线 complier：playground](http://www.typescriptlang.org/play/index.html)
- [浅谈 TypeScript: typescript-book](https://github.com/welearnmore/typescript-book)
- [typescript-tutorial: TypeScript 入门教程]( https://github.com/xcatliu/typescript-tutorial)

- [深入理解 TypeScript](https://jkchao.github.io/typescript-book-chinese/)
