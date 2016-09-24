---
title: python删除mysqlbinlog日志中的删除语句实现
date: 2016-09-25 10:50:36
desc: 
tags: python
---

昨天写的mysqlbinlog处理数据的帖子, 最后有个问题, 就是数据备份出来的大小有好几G, 单纯的用编辑器编写, 毫无疑问, 不行的. 今天用python写了个脚本, 实现它.

<!-- more -->

### 主要思路

* 检测**BINLOG**到**BINLOG**段的行数放入一个集合中, 变量匹配, 如果集合中有删除`delete`, 字段, 则不写入到结果文件中.

* 直接贴码了, **python** 好久没用了, 代码写的一般, 欢迎拍砖.

    ```python
        #!/usr/bin/env python
        #-*- encoding: utf-8 -*-
        # author luowen<bigpao.luo@gmail.com>

        import argparse, re, time
        from functools import reduce


        class CutDelete():
            """
                将backup.sql中的delete语句删除处理脚本
            """
            PREFIX = "BINLOG '"
            listOfLine = []

            def __init__(self, inputFilename, outputFilename):
                self.inputFilename = inputFilename
                self.outputFilename = outputFilename

            def writeOutputFile(self):
                fileHandle = open(self.outputFilename, 'a+', encoding="utf-8")
                if len(self.listOfLine) > 0:
                    rePatternOfDelete = re.compile(".*DELETE.*", re.IGNORECASE)
                    isSkip = False
                    for line in self.listOfLine:
                        if rePatternOfDelete.match(line):
                            isSkip = True
                            break;
                    if not isSkip:
                        fileHandle.writelines(self.listOfLine)
                    self.listOfLine = [] # reset list of line
                fileHandle.close()

            def main(self):
                with open(self.inputFilename, "r", encoding="utf-8") as fileHandle:
                    for line in fileHandle:
                        if line.startswith(self.PREFIX):
                            self.writeOutputFile()
                        self.listOfLine.append(line)
                return True

        class InputArgsParser():
            """
                获取输入文件, 和写出文件
            """

            def __init__(self):
                self.parser = argparse.ArgumentParser(description="获取输入文件和输出文件")

            def getArgs(self):
                self.parser.add_argument("--input", type=str, required=True, help="需要处理的文件")
                self.parser.add_argument("--output", type=str, required=True, help="处理后的文件名")
                return self.parser.parse_args()



        if __name__ == "__main__":

            inputArgsParser = InputArgsParser()
            args =  inputArgsParser.getArgs()

            doCutDeleteObj = CutDelete(args.input, args.output)
            resultSet = doCutDeleteObj.main()
    ```

* 有兴趣可以使用php的`yield, yield -> send` 实现, 也是不错的选择!
