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
float xvariance(int input){
	float avg = 0;
	float variance = 0;

	int n = npoints(input);

	for(int i = 0; i < n; i++){
		float x = point(input, "P", i)[0];
		avg += x;
	}

	//the mean of our data set
	avg = avg / float(n);

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

it's probably easier to see in code....

```c
float covariance(int input){
	float xavg = 0;
	float yavg = 0;
	float covariance = 0;

	int n = npoints(input);

	for(int i = 0; i < n; i++){
		vector p = point(input, "P", i);
		float x = p.x;
		float y = p.y;
		xavg += x;
		yavg += y;
	}

	//the means of our two dimensions
	xavg = xavg / float(n);
	yavg = yavg / float(n);

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

@covariance = covariance(0);
```

Boom, that's it. But we work in 3d, and that's a two dimensional variance anylysis. Sooooo how do we make it work in 3d. Well quite simply we build of the covariance on each axis matrix. 

Meaning our matrix will look like:

```c
[covar(x, x), covar(x, y), covar(x,z)]
[covar(y,x), covar(y, y),  covar(y,z)]
[covar(z,x), covar(z, y), covar(z, z)]
```

The above matrix is really interesting to me for a few reasons, first and most importantly you'll notice the diagnol of the matrix, is actually just the variance on each individual dimension. Another interesting property is that the matrix is symmetric along the daignol, meaning if you take the transpose of it, the values should stay the exact same. Neat.

Okay so let's modify our code to work for a 3 dimensional covariance matrix. Im going to change the syntax of a few things to make it easier to build. One key one being, the values we want to find the covariance for, will be stored in an array prior to calculating the matrix, that way we don't have to search a shit ton of times.


```c
float covariance(vector a[]; int c, d){
	//vector a[] is our input data set
	//int c, d are the vector components we want to analyze

    float cvr;
    float avgx, avgy;
    int i, j;
    
    
    for(j = 0; j < len(a); j++){
        avgx += a[j][c];
        avgy += a[j][d];
    }
    
    avgx /= len(a);
    avgy /= len(a);
    
    for(i = 0; i < len(a); i++){
        float x = (a[i][c] - avgx);
        float y = (a[i][d] - avgy);
        cvr += x * y;
        }
    
    cvr /= len(a) - 1;    
    return cvr;
}


//build our array of point positions
vector pos_array[];
for(int i = 0; i < npoints(0); i++){
    vector p = point(0, "P", i);
    append(pos_array, p);
}



//create our symmetric covar matrix
vector row1, row2, row3;
row1 = set(covar(pos_array, 0, 0), covar(pos_array, 0, 1), covar(pos_array, 0, 2));
row2 = set(covar(pos_array, 1, 0), covar(pos_array, 1, 1), covar(pos_array, 1, 2));
row3 = set(covar(pos_array, 2, 0), covar(pos_array, 2, 1), covar(pos_array, 2, 2));

matrix3 m = set(row1, row2, row3);
3@m = m;
```

BOOM FUCKING EASY MATES. 


Alright now comes the actually difficult part. It's not difficult in implementation, but it is difficult to understand. We are now entering the world of principal component analysis (PCA for short). PCA at it's core is a tool that helps us as wizards, determine the principal components of a given input. Below is a simple diagram showing the principal curvatures (slightly different) of a given surface. The principal components, are the directions of flow that most accuractely describe the surface. That's still really hard to understand... An even easier way to describe it, is if we want an oriented, 3d bounding box, it needs to be oriented to the principal components of the input mesh.... fuck.


In math terms to solve this, we need to find the eigen vectors of our covariance matrix. Or in other words, what are the vectors that when transformed by the covariance matrix, do nothing outside of scaling uniformly. Meaning, what is the axis that has the most variance (found through power iteration), what axis has the least variance, and then from there the final axis is the cross product of the other two, as they should be orthogonal to each other.


Yeah. Like i said it's difficult, and like honestly it just doesn't make that much since till you try it. If you want more insite into eigenvectors, peep this video by 3blue1brown:

And then finally the code:

```
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

