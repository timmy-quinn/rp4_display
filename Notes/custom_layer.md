# Building a custom layer


A layer is a repo which contains related metadata. Tells how to build a layer. 
A layer contains recipes. Recipes describe how to create a particular package. 


finding variable names from a recipe/package
```
bitbake -e recipe-name | grep ^variable_name= 
```

### Variable assignnemnts  

?= sets default value which can be overwritten. The first of this type will be used 
??= : weak assignment. It is a weak assignment. If multiple of this type are 
done, the last of this type will be used


= : simple assignment. It requires "" and spaces are significant 
:= : immediate variable expansion. Value is immediately expanded

+= appends. inserts a space between current and appended
=" prepends. inserts space between current and prepended 

.= :appends with no space. 
=. :prepends with no space

:append : appends with no space. Applied at expansion time rather than immediately 
:prepend : prepends with no spaces. Applied at expansion 
:remove : removes a value from a list. Affects all occurences. Done at end of 
variable expansion

### 
some examples 

**Example 1**
```
A = "foo"
B = "${A}"
A = "bar"
```

Result 
```
$ btibake -e example | grep ^B= 
B="bar"
```

**Example 2**
```
A ?= "foo"
B := "${A}"
A = "bar"
```

```
$ btibake -e example | grep ^B= 
B="foo"
```




