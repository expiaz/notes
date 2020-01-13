# $@ = target file
# $< = first dependency
# $^ = all dependencies
#
# dependency : file or rule name
# rulename: file or ascii name
# command: shell command
# ----------------------------------
# rulename: dependency1 dependency2
# 	command
# $@ => rulename
# $< => dependency1
# $^ => dependency1 dependency2