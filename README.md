# houdini_black_magic
some serious voodo bruh. 



This is honestly like stuff you should only whip out when trying to impress a honey u kno? SLAP DOWN TWO WRANGLES BOOYYYYYYYYYYYYY. It's easy to miss that the VEXpression section of a wrangle is actually just a parameter like any string parameter, just thiccer. So like any string parameter, you can use hscript orr python. Take a second and let that sink in, python.

DROP THIS IN THIS TOP WRANGLE BOY:
```
float r = rand(@ptnum) * 3;
r = floor(r);
if(r == 0) s@class = "a";
if(r == 1) s@class = "b";
if(r == 2) s@class = "c";

f@val;
```


#MOVING ON TO THE SECOND WRANGLE. 

Right click you vexpression and drop down a keyframe and set the vexpression language to python. BOOM we're in.

Remember i said voodoo shit, when i said let that sink in, nothing probably sank in youre drugged the fuck up jake what r u doing. We're going to do some ultra dumb shit, let's use python to write VEX. Like any python expression you need to return your result. So any string we return that contains vex code will get executed by this parameter.

Let's start with this: 



SET THE BOTTOM ONE TO RUN IN PYTHON MODE
```
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
