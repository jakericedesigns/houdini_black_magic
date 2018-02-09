# houdini_black_magic
some serious voodo bruh. 



This is honestly like stuff you should only whip out when trying to impress a honey u kno? SLAP DOWN TWO WRANGLES BOOYYYYYYYYYYYYY. It's easy to miss that the VEXpression section of a wrangle is actually just a parameter like any string parameter, just thiccer. So like any string parameter, you can use hscript orr python. Take a second and let that sink in, python.

DROP THIS IN THIS TOP WRANGLE BOY:
```c
float r = rand(@ptnum) * 3;
r = floor(r);

if(r == 0) s@class = "a";
if(r == 1) s@class = "b";
if(r == 2) s@class = "c";

f@val;
```


#MOVING ON TO THE SECOND WRANGLE. MAKE IT A DETAIL BOI

Right click you vexpression and drop down a keyframe and set the vexpression language to python. BOOM we're in. 

We're going to do some ultra dumb shit, let's use python to write VEX. Like any python expression you need to return your result. So any string we return that contains vex code will get executed by this parameter.

Let's start with this: 



SET THE BOTTOM ONE TO RUN IN PYTHON MODE
```py
node = hou.pwd()
input = node.inputs()
geo = input[0].geometry()
points = geo.points()

#define our functions
out = """
float aa(float i){ 
    return pow(i, 2); 
}
float ab(float i){ 
    return exp(i); 
}
float ac(float i){ 
    return sqrt(i);
} \n"""

out += "float val;\n"

for i, point in enumerate(points):
	#class attribute drives which function is selected. 
    out += "val = a%s(10);\n" % (point.attribValue("class"))
    out += "setpointattrib(0, 'val', %d, val);\n" % i

return out
   
```



WHAT DOES THIS EVEN DOOOOOO. ANSWER: NOTHING!. all it does is show how you can dynamically complete VEX expressions with python, notice how im filling in the suffix of the function name in the first line of code in the for loop?
This line specifically: `out += "val = a%s(10);\n" % (point.attribValue("class"))`

If you want to see the vex this outputs just click on the VEXpression parm and boom you can see what this outputs. wowowowowowow.
I only have 4 points in this example, so here's what that would look like when it gets parsed:

```c
    float aa(float i){ 
        return pow(i, 2); 
    }
    float ab(float i){ 
        return exp(i); 
    }
    float ac(float i){ 
        return sqrt(i);
    } 
float val;
val = ab(10);
setpointattrib(0, 'val', 0, val);
val = aa(10);
setpointattrib(0, 'val', 1, val);
val = ac(10);
setpointattrib(0, 'val', 2, val);
val = aa(10);
setpointattrib(0, 'val', 3, val);
```
BAM DONE. NEXT!
