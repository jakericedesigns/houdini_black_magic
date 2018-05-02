# houdini_black_magic
some serious voodo bruh. 
This is going to eventually become a lot of contributions to the advanced section on the community site :o
Right now it's quite unprofessionally written and has not been proofread at all.



This is honestly like stuff you should only whip out when trying to impress a honey u kno? 

## EXAMPLE 1 - WRITING VEX WITH PYTHON
SLAP DOWN TWO WRANGLES BOOYYYYYYYYYYYYY. It's easy to miss that the VEXpression section of a wrangle is actually just a parameter like any string parameter, just thiccer. So like any string parameter, you can use hscript orr python. Take a second and let that sink in, python.

### IN WRANGLE #1 RUNNING OVER POINTS:
```c
float r = rand(@ptnum) * 3;
r = floor(r);

if(r == 0) s@class = "a";
if(r == 1) s@class = "b";
if(r == 2) s@class = "c";

f@val;
```


### IN WRANGLE #2 RUNNING OVER DETAILS:

Right click you vexpression and drop down a keyframe and set the vexpression language to python. BOOM we're in. 
Like any python expression you need to return your result. So any string we return that contains vex code will get executed by this parameter.

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



### WHAT DOES THIS EVEN DOOOOOO. ANSWER: NOTHING!. 
All it does is show how you can dynamically complete VEX expressions with python, notice how im filling in the suffix of the function name in the first line of code in the for loop?

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




## EXAMPLE 2 - The Covariance Matrix/PCA

Hello my dudes. Prior knowledge needed for this is defo like linear algebra, and maybe a bit of statistics. Covariance is a concept that comes up all over the place in the land of statistics and higher levels of math. It's a really nice tool for solving all kinds of problems, including finding an oriented bounding box for a given point set, adding normals to a point cloud for relighting, among many others.

If you don't care to understand how this works, just scroll to the bottom, but just know that i h8 u for it :(

Also I'm writing this in VEX (like an easier C), however I'm trying to code it in the most readable way, as this can be easily implemented in pretty much any language. Though i would guess you have access to libraries for this stuff...

At the crux of a variance, and likewise covariance is the idea of finding the mean (AKA the average value) of a data set. In the case of something like houdini there are plenty of ways averaging the values of all sorts of different operations, let's start this off by working with the average X position of the classic pighead.

### So let's drop down a pighead and then a wrangle in detail mode.

```c
//THIS GOES INTO A DETAIL WRANGLE
float xavg(int input){
	float avg = 0;
	for(int i = 0; i < npoints(input); i++){
		float x = point(input, "P", i)[0];
		avg += x;
	}
	return avg / float(npoints(input));
}

@xavg = xavg(0);
```

The above code will return the average X position of all the point coming in from input 0 of our wrangle. Neat. Since this is the advanced section of the wiki, that should be a refresher.

Now how does that relate to variance. First it's important to understand why I used just the X component in my above mean calculation, when really you can quite easily average vector values. Variance by definition is a one dimensional calculation. It's actually an incredibly simple concept, all variance is, is the sum of all the SQUARED distances from a given point in the data set, to the mean or of that data set, all divided by the length of the data set (to normalize the values).

You might be wondering why we're using the squared distance instead of the euclidean distance, since variance is only a distance test, keeping the distance squared helps excentuate the deviation between different points to the average. It'd still probably work if you took the euclidean distance, but that'd add extra computation time that's unnecessary. And if you're confused as to why we square the value, it's because it prevents us from having any negative distances, which would be stupid. 

So now instead of doing xavg, let's write the function for xvariance!


```c
//THIS GOES INTO A DETAIL WRANGLE
float xvariance(int input){
	float avg = xavg(input);
	float variance = 0;
	
	int n = npoints(input);
	
	for(int j = 0; j < n; j++){
		float x = point(input, "P", i)[0];
		x = x - avg;
		x = x * x;

		variance += x;
	}

	return variance / (float(n) - 1);
}

@xvariance = xvariance(0);
```

But wait, why am i subtracting one from our total point count when returning variance, when i dont do that when i return average. To be honest, that's probably the hardest part of this whole thing to explain (minus eigenvectors), and I'd rather let true math wizards explain that, so if you're curious, check the bottom of page three of this PDF: http://www.cs.otago.ac.nz/cosc453/student_tutorials/principal_components.pdf

Alright bubs we're inching ever closer, now what is covariance. Covariance is stupid simple, since I said variance is a one dimensional operation, covariance is the same thing but on two axix. By that I mean we should now calculate the mean of the x axis and the mean of the y axis, and then find the combined* distance from any of the given data points.

I say combined, because instead of squaring the delta (distance) from x to the average, you multiply the x delta by the y delta.

In order to make things easier, im going to also update the `xavg` function to a more general `pos_avg()` function. 

This is all probably easier to see in code....



```c
//THIS GOES INTO A DETAIL WRANGLE
vector pos_avg(int input){
	vector avg = 0;
	for(int i = 0; i < npoints(input); i++){
		vector a = point(input, "P", i);
		avg += a;
	}
	return avg / float(npoints(input));
}


float covariance_xy(int input){
	float xavg = pos_avg(input)[0];
	float yavg = pos_avg(input)[1];
	float covariance = 0;

	int n = npoints(input);

	for(int j = 0; j < n; j++){
		vector p = point(input, "P", i);
		
		float x = p.x;
		float y = p.y;

		x = x - xavg;
		y = y - yavg;

		//x * y instead of x * x
		x = x * y;

		covariance += x;
	}

	return covariance / (float(n) - 1);
}

@covariance = covariance_xy(0);
```

Boom, that's it, you have now mastered basic covariance.

### COVARIANCE MATRICES AND DOING THIS WHOLE TING SMARTER 
We work in 3d, and that's a two dimensional variance anylysis. So what we need is a matrix of covariance.

```c
//a sample covariance matrix
[covar(x, x), covar(x, y), covar(x,z)]
[covar(y,x), covar(y, y),  covar(y,z)]
[covar(z,x), covar(z, y), covar(z, z)]
```

The above matrix is really interesting for a few reasons, but the most important one for us is the fact that its symmetric along the diagonal. Meaning if we rethink our above code in a more clever way, we can build `OUR FULL COVARIANCE MATRIX` in a way that's so much more Algebraic and Houdini way in execution.

First step is to find the average position of our mesh, for that a simple attribute promote from point `P` to a detail attribute will work.
![Attribute Promotion](./img/attrib_promote.png)

Next we need to get the delta from our position to the average so we can start building the covariance matrix

```c
//THIS GOES INTO A POINT WRANGLE
vector delta = v@P - detail(0, "avg_pos");
```


The next part is where things get fun, if you think about the definition of the outer product operator, you might come to realize that if you take the outer product of a vector and itself you're left with a symmetric matrix. Here's a visual to help:

```c
vector A;
outerproduct(A, A) ==

[A.x * A.x, A.x * A.y, A.x * A.z]
[A.y * A.x, A.y * A.y, A.y * A.z]
[A.z * A.x, A.z * A.y, A.z * A.z]
```

Well that's convenient, since we already know a core part of variance is that it's the squared distance to the mean, over the number of samples minus 1. And if we substitute our delta (the distance from our sample to the mean) for A in the above outerproduct example, you'll see it creates a matrix of partially solved variance results. The final things we need to do to make it correct are:
* sum this matrix up over all points in the mesh, like we do in our previous examples!
* then, we can divide out the number of points minus one from the sum, as discussed before.


```c
//THIS GOES INTO A POINT WRANGLE
vector delta = v@P - detail(0, "avg_pos");
matrix3 covar = outerproduct(delta, delta);

setdetailattrib(0, "covar", covar, "add");

```

```c
//THIS GOES INTO A DETAIL WRANGLE
3@covar /= float(npoints(0)) - 1.;
```

That's so much cleaner. I love you Houdini.


## EIGEN VECTORS ARE FUNKY STUFF 

Alright now comes the actually difficult part. It's not difficult in implementation, but it is difficult to understand. We are now entering the world of principal component analysis (PCA for short). PCA at it's core is a tool that helps us as wizards, determine the principal components of a given input. Below is a simple diagram showing the principal curvatures (slightly different) of a given surface. The principal components, are the directions of flow that most accuractely describe the surface. That's still really hard to understand... An even easier way to describe it, is if we want an oriented, 3d bounding box, it needs to be oriented to the principal components of the input mesh.... fuck.


In math terms to solve this, we need to find the eigen vectors of our covariance matrix. Or in other words, what are the vectors that when transformed by the covariance matrix, do nothing outside of scaling uniformly. Or put even more simply (and shamelessly stolen from wikipedia) in the geometric context, an eigenvector tells you the direction in which stretching will occur, and the eigenvalues, tells you the amount of stretch applied in that given direction. However, this assumes that our matrix has non imaginary eigenvalues. Negative Eigenvalues reverse the direction of stretch. 

So in the context of our covariance matrix, what is the axis that has the most variance (found through power iteration), what axis has the least variance, and then from there the final axis is the cross product of the other two, as they should be orthogonal to each other.

Yeah. Like i said it's difficult, and like honestly it just doesn't make that much since till you try it. If you want more insite into eigenvectors, peep this video by 3blue1brown:

And then finally the code:

```c
//power iteration to find the largest eigen vector
//https://en.wikipedia.org/wiki/Power_iteration

function vector[] p_iter(matrix3 m){
    vector bk = rand(determinant(m) * rand(determinant(m)));
    vector w = rand(determinant(m) * rand(determinant(m)));
    vector array[];
    for(int i = 0; i < 100; i++){
        //power iteration
        bk *= m;
        bk = normalize(bk);
        
        //inverse iteration
        w *= invert(m); 
        w = normalize(w);
        }
    
    push(array, bk);
    push(array, w);
    return array;
}

v@x = p_iter(3@m)[0];
v@y = p_iter(3@m)[1];
```

And that's pretty much it....

I donno it's kinda confusing too be honest, and I'm sorry. But like i said this does really involve a lot of deep linear algebra concepts. And it's okay for it to not make sense, this shit can take a minute for it to truly click. I think the best way is to just put it into practice and fuck around with the code, the best way to figure out why something was done is to try to figure out the problem yourself. 

