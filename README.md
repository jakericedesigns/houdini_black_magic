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

Hello my dudes. Prior knowledge needed for this is defo like linear algebra, and maybe a bit of statistics. Covariance is a concept that comes up all over the place in the land of statistics and higher levels of math. With the extention of PCA (principal component analysis) we can turn covariance into a powerhouse of a tool that can be used for solving all types of problems, from adding normals to a point cloud, to creating an oriented bounding box for a given object.

If you don't care to understand how this works, just scroll to the bottom, but just know that i h8 u for it :(

Like always we'll be using Houdini and it's internal scripting language, VEX for this.

To understand Covariance, you must first understand variance. At it's core variance is all about finding the squared delta (difference) of a variable to it's mean. Or in more readable terms, what's the average distance from any sample in a data set, to it's average.

In a 1 dimensional system, where X is some random variable with *n* samples, variance is defined as this:

![Variance Equation](./img/variance_equation.svg)

Where ![mu](./img/mu.svg) is the mean of our data set defined by:

![1d Mean Equation](./img/mean_equation_1d.svg)

As a refresher, let's start by just doing the variance of the X axis of a given input object.

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

Again since the prior knowledge asks for linear algebra, I'll assume you understand how to calculate the average of a data set...

From here all we need to is calculate the squared distance from a given sample to the average, sum that up over all samples and divide out our sample size to normalize it!

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

	return variance / float(n);
}

@xvariance = xvariance(0);
```



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

//WHERE A and B are the vector components you want to test against
float covar(int input, int a, int b){
	float a_avg = pos_avg(input)[a];
	float b_avg = pos_avg(input)[b];
	float covariance = 0;

	int n = npoints(input);

	for(int j = 0; j < n; j++){
		vector p = point(input, "P", i);
		
		float x = p[a];
		float y = p[b];

		x = x - a_avg;
		y = y - b_avg;

		//x * y instead of x * x
		x = x * y;

		covariance += x;
	}

	return covariance / (float(n) - 1);
}
//covariance between the X and Y axis
@covariance = covariance(0, 0, 1);
```

But wait, why am i subtracting one from our total point count when returning covariance and not variance... To be honest, that's probably the hardest part of this whole thing to explain (minus eigenvectors), and I'd rather let true math wizards explain that, so if you're curious, check the bottom of page three of this PDF: http://www.cs.otago.ac.nz/cosc453/student_tutorials/principal_components.pdf


But we work in 3d, and that's a two dimensional variance anylysis. So what we need is a matrix of covariance. The below code block is what a basic covariance matrix looks like with the functions we defined above.

```c
//a sample covariance matrix

int x = 0, y = 1, z = 2;

matrix3 covar = 
set(covar(0, x, x), covar(0, x, y), covar(0, x, z),
 	covar(0, y, x), covar(0, y, y), covar(0, y, z),
	covar(0, z, x), covar(0, z, y), covar(0, z, z))
```

### COVARIANCE MATRICES AND DOING THIS WHOLE TING SMARTER 

The above matrix is really interesting for a few reasons, but the most important one for us is the fact that its symmetric along the diagonal. Meaning if we rethink our above code in a more clever way, we can build **OUR FULL COVARIANCE MATRIX** in a way that's so much more algebraic and Houdini way in execution.

First step is to find the average position of our mesh, for that a simple attribute promote from point `P` to a detail attribute will work.

![Attribute Promotion](./img/attrib_promote.png)

Next we need to get the delta from our position to the average so we can start building the covariance matrix

```c
//THIS GOES INTO A POINT WRANGLE
vector delta = v@P - detail(0, "avg_pos");
```


The next part is where things get fun, if you think about the definition of the outer product operator, you might come to realize that if you take the outer product of a vector and its' transpose you're left with a symmetric matrix. Here's a visual to help:

```c
vector A;
outerproduct(A, A) ==

[A.x * A.x, A.x * A.y, A.x * A.z]
[A.y * A.x, A.y * A.y, A.y * A.z]
[A.z * A.x, A.z * A.y, A.z * A.z]
```

Well that's convenient, since we already know a core part of variance is that it's the squared distance to the mean, over the number of samples minus 1. And if we substitute our delta (the distance from our sample to the mean) for A in the above outerproduct example, you'll see it creates a matrix of partially solved variance results. The final things we then need to do to make it correct are:
* sum this matrix up over all points in the mesh, like we do in our previous examples!
* then, we can divide out the number of points minus one from the sum, as discussed before.


```c
//THIS GOES INTO A POINT WRANGLE
vector delta = v@P - detail(0, "avg_pos");
matrix3 covar = outerproduct(delta, delta);

//this is the exact same thing as an attribute promotion in add mode!
setdetailattrib(0, "covar", covar, "add");

```

![Build Covar](./img/build_covar.png)


```c
//THIS GOES INTO A DETAIL WRANGLE
3@covar /= float(npoints(0)) - 1.;
```

![Normalize Covar](./img/normalize_covar.png)

That's so much cleaner. I love you math.


### EIGEN VECTORS ARE FUNKY STUFF 

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

