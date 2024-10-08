# Javira Programming Language
> **The Supercool Transpiled Language that’s More Fun than a Barrel of Monkeys! 🐒**

## What is Javira?
Javira is a teeny-tiny programming language born from the mind of a caffeinated developer and coded in C. It magically transforms into Java! 🎩✨

> Why Javira? Well, it's designed to be a blast, just like Kotlin—only better! With those parentheses (`(` and `)`) everywhere, you'll feel like you're coding in a cozy, curly-hair salon. Yes, I adore those parentheses! 😍

**Example Taste**
```lisp
;; Ready to dive into the world of Fibonacci with Javira?
;; This recursive function calculates Fibonacci numbers, because who doesn't love a bit of recursion?

(func fib (param i int) (result int)
	;; Base case: If `i` is 0 or 1, return `i`
	;; Because Fibonacci numbers start with 0 and 1, obviously!
	(if (<= i 1) (then
		(return i)
	))

	;; Recursive case: The magic happens here
	;; Add up the results of fib(i-1) and fib(i-2) like a Fibonacci sandwich
	(return (+
		(call fib (- i 1)) ;; Compute Fibonacci number for (i-1)
		(call fib (- i 2)) ;; Compute Fibonacci number for (i-2)
	))
)

;; Main function to show off the Fibonacci awesomeness
(func main (result int)
	;; Calculate the 9th Fibonacci number
	;; Because why not? You can totally change this index and impress your friends!
	(let x int (call fib 9))

	;; Print the result, so you can show off your Fibonacci-finding skills
	(print x)

	;; Return 0 like a boss, indicating everything went swimmingly
	(return 0)
)
```

> **As you can see, writing in Javira is like crafting a high-level abstract syntax tree with a splash of whimsical fun! 🌳🌟**

## Setup
> Just put `this dir` in your %%PATH%%
```ps
PS G:\> ls


    Directory: G:\


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         8/31/2024   6:53 AM                std
-a----         8/31/2024   7:00 AM         260884 Javira.exe
-a----         8/31/2024   7:18 AM          13694 README.md
```

# Cli Tool

**BUILD**
```ps
PS G:\Dev\javira\src> javira --build ../examples/helloworld.jv       
@info transpiling '../examples/helloworld.jv'...
@info running preprocessor...
@info constructing syntax tree...
@info compiling syntax tree...
@info generating target code: out.java
@info freeing 5038 bytes (133 objects).
```

**RUN**
```ps
PS G:\Dev\javira\src> javira --run ../examples/helloworld.jv
@info transpiling '../examples/helloworld.jv'...
@info running preprocessor...
@info constructing syntax tree...      
@info compiling syntax tree...
@info generating target code: out.java 
@info freeing 5038 bytes (133 objects).
@info compiling Java code...
@info running Java code...
hello world!
@info cleaning up...
```

# Features

- Javira is strongly statically typed. No surprises here!
- Built-in support for arrays, hashtables, and structs. Everything you need to store stuff!
- C-like macros to play with. Customize your code for different targets and embed code like a pro.
- The syntax is as simple as pie: tokens in parentheses `()`, with the first token always being a keyword/operator. A delightful 50 keywords to choose from!
- No OOP here! You get structs to group data, but no blending methods. It’s pure data power! 💪
- Javira doesn’t have a boolean type: zero means false, non-zero means true. It’s that simple!
- Not garbage-collected, but fear not! It has tools to help with memory management, making those pesky leaks less likely.

**DETAILED EXAMPLE**
> Borrowed from another whimsical programming language! 🌟

```lisp
;; raycast.jv
;; bare-minimum raycaster demo
;;
;; - ray-triangle intersection
;; - "n-dot-l" shading
;; - face normals only
;; - no trees/partitioning for acceleration
;; - includes a dodecahedron example
;; - ascii art for plotting renders

(@include math)

;; dimension of camera
(@define W      80)
(@define H      40)
(@define WxH  3840)
(@define FOCAL 100)

;; datastructure for a ray
(struct ray
	(let o (vec 3 float)) ; origin
	(let d (vec 3 float)) ; direction
	(let tmin float)      ; t parameter extremas
	(let tmax float)
)

;; datastructure for a mesh
(struct mesh
	(let vertices  (arr (vec 3 float)))  ; 3d coordinates
	(let faces     (arr (vec 3 int  )))  ; winding
	(let facenorms (arr (vec 3 float)))
)

;; ========== vector math utilities ========== 

;; subtract
(func v_sub 
	(param u (vec 3 float))
	(param v (vec 3 float))
	(result  (vec 3 float))

	(return (alloc (vec 3 float)
		(- (get u 0) (get v 0))
		(- (get u 1) (get v 1))
		(- (get u 2) (get v 2))
	))
)

;; cross product
(func v_cross
	(param u (vec 3 float))
	(param v (vec 3 float))
	(result  (vec 3 float))
	(return (alloc (vec 3 float)
		(-
			(* (get u 1) (get v 2))
			(* (get u 2) (get v 1))
		)(-
			(* (get u 2) (get v 0))
			(* (get u 0) (get v 2))
		)(-
			(* (get u 0) (get v 1))
			(* (get u 1) (get v 0))
		)
	))
)

;; dot product
(func v_dot
	(param u (vec 3 float))
	(param v (vec 3 float))
	(result float)

	(return (+
		(* (get u 0) (get v 0))
		(* (get u 1) (get v 1))
		(* (get u 2) (get v 2))
	))
)

;; resize by a scalar
(func v_scale
	(param u (vec 3 float))
	(param x float)
	(result (vec 3 float))
	
	(return (alloc (vec 3 float)
		(* (get u 0) x)
		(* (get u 1) x)
		(* (get u 2) x)
	))
)

;; magnitude (euclidean norm)
(func v_mag
	(param v (vec 3 float))
	(result float)
	
	(return (call sqrt
		(+
			(* (get v 0) (get v 0))
			(* (get v 1) (get v 1))
			(* (get v 2) (get v 2))
		)
	))
)

;; normalize to unit vector *in-place*
(func normalize
	(param v (vec 3 float))
	
	(let l float (call v_mag v))
	(set v 0 (/ (get v 0) l))
	(set v 1 (/ (get v 1) l))
	(set v 2 (/ (get v 2) l))
)

;; determinant
(func det
	(param a (vec 3 float))
	(param b (vec 3 float))
	(param c (vec 3 float))
	(result float)

	(local d (vec 3 float) (call v_cross a b))
	(let e float (call v_dot d c))
	(return e)
)

;; ========== ray calculations ========== 

;; create new ray
(func new_ray
	(param ox float) (param oy float) (param oz float)
	(param dx float) (param dy float) (param dz float)
	(result (struct ray))
	
	(let r (struct ray)  (alloc (struct ray)))
	(let o (vec 3 float) (alloc (vec 3 float)
		ox oy oz
	))
	(let d (vec 3 float) (alloc (vec 3 float)
		dx dy dz
	))
	(call normalize d)
	(set r o o)
	(set r d d)
	(set r tmin 0.0)
	(set r tmax INFINITY)
	(return r)
)

;; free allocated ray
(func destroy_ray
	(param r (struct ray))
	(free (get r o))
	(free (get r d))
	(free r)
)

;; ray-triangle intersection
(func ray_tri
	(param r  (struct ray) )
	(param p0 (vec 3 float)) 
	(param p1 (vec 3 float))
	(param p2 (vec 3 float))
	(result float)

	(local e1 (vec 3 float) (call v_sub p1 p0))
	(local e2 (vec 3 float) (call v_sub p2 p0))
	
	(local s  (vec 3 float) (call v_sub   (get r o) p0))
	(local _d (vec 3 float) (call v_scale (get r d) -1))
	
	(let denom float (call det e1 e2 _d))
	
	(if (= denom 0) (then
		(return INFINITY)
	))
	
	(let u float (/ (call det s e2 _d) denom))
	(let v float (/ (call det e1 s _d) denom))
	(let t float (/ (call det e1 e2 s) denom))
	
	
	(if (||
		(< u 0) (< v 0)
		(< (- 1 (+ u v)) 0)
		(< t (get r tmin)) (> t (get r tmax))) (then

		(return INFINITY)	
	))
	
	(set r tmax t)
	(return t)
)

;; ray-mesh intersection
;; naive triangle-by-triangle checking 
;; -- for better speed, replace with BVH, k-d tree etc.
(func ray_mesh
	(param r (struct ray ))
	(param m (struct mesh))
	(param l (vec 3 float))
	(result float)

	(let dstmin float INFINITY)
	(let argmin int -1)
	(for i 0 (< i (# (get m faces))) 1 (do
		(let a (vec 3 float) (get m vertices (get m faces i 0)))
		(let b (vec 3 float) (get m vertices (get m faces i 1)))
		(let c (vec 3 float) (get m vertices (get m faces i 2)))
		(let t float (call ray_tri r a b c))
		(if (< t dstmin) (then
			(set dstmin t)
			(set argmin i)
		))
	))
	(if (|| (< argmin -1) (= dstmin INFINITY)) (then
		(return 0.0)
	))
	
	(let n  (vec 3 float) (get m facenorms argmin))

	(let ndotl float (call v_dot n l))
	(return (+ (call fmax ndotl 0) 0.1))
)

;; ========== mesh calculations ========== 

;; add vertex to mesh
(func add_vert
	(param m (struct mesh))
	(param x float)
	(param y float)
	(param z float)
	
	(insert (get m vertices) (# (get m vertices))
		(alloc (vec 3 float) x y z)
	)
)

;; add face to mesh
(func add_face
	(param m (struct mesh))
	(param a int)
	(param b int)
	(param c int)
	
	(insert (get m faces) (# (get m faces))
		; hacks specific to .obj files:
		; -1 because in .obj format index starts at 1
		; (a,c,b) because some .obj's wind the wrong way
		(alloc (vec 3 int) (- a 1) (- c 1) (- b 1))
	)
)

;; pre-compute the face normals
(func calc_facenorms (param m (struct mesh))
	(for i 0 (< i (# (get m faces))) 1 (do
		(let a (vec 3 float) (get m vertices (get m faces i 0)))
		(let b (vec 3 float) (get m vertices (get m faces i 1)))
		(let c (vec 3 float) (get m vertices (get m faces i 2)))
		
		(local e1 (vec 3 float) (call v_sub a b))
		(local e2 (vec 3 float) (call v_sub b c))

		(let n  (vec 3 float) (call v_cross e1 e2))
		(call normalize n)
		(insert (get m facenorms) (# (get m facenorms)) n)
	))
)

;; translate mesh
(func move_mesh 
	(param m (struct mesh)) 
	(param x float) (param y float) (param z float)
	(for i 0 (< i (# (get m vertices))) 1 (do
		(set (get m vertices i) 0 (+ (get m vertices i 0) x))
		(set (get m vertices i) 1 (+ (get m vertices i 1) y))
		(set (get m vertices i) 2 (+ (get m vertices i 2) z))
	))
)

;; free allocated mesh
(func destroy_mesh
	(param m (struct mesh))
	(for i 0 (< i (# (get m vertices))) 1 (do
		(free (get m vertices i))
	))
	(free (get m vertices))
	(for i 0 (< i (# (get m faces))) 1 (do
		(free (get m faces i))
	))
	(free (get m faces))
	(for i 0 (< i (# (get m facenorms))) 1 (do
		(free (get m facenorms i))
	))
	(free (get m facenorms))
	(free m)
)

;; render a mesh and print ASCII art
(func render (param m (struct mesh)) (param light (vec 3 float))
	(local pix (vec @WxH float) (alloc (vec @WxH float)))
	
	(call normalize light)
	
	(let palette str "`.-,_:^!~;r+|()=>l?icv[]tzj7*f{}sYTJ1unyIFowe2h3Za4X%5P$mGAUbpK960#H&DRQ80WMB@N")
	(let lo float INFINITY)
	(let hi float 0)
	(for y 0 (< y @H) 1 (do
		(for x 0 (< x @W) 1 (do
			(let fx float (/ (- x (/ @W 2.0)) 2.0 ))
			(let fy float (- y (/ @H 2.0)))
			(let r (struct ray) 
				(call new_ray 0 0 0 fx fy @FOCAL)
			)
			(let gray float (call ray_mesh r m light))
			(set hi (call fmax gray hi))
			(if (> gray 0) (then
				(set lo (call fmin gray lo))
			))
			(set pix (+ (* y @W) x) gray)
			(call destroy_ray r)
		))
	))
	
	(local s str (alloc str))
	(for y 0 (< y @H) 1 (do
		(for x 0 (< x @W) 1 (do
			(let gray float (get pix (+ (* y @W) x)))
			(if (<> gray 0) (then
				(set gray (/ (- gray lo) (- hi lo) ))
				(let ch int (get palette (cast (* gray 78) int)))
				(<< s ch)
			)(else
				(<< s ' ')
			))
			
		))
		(<< s "\n")
	))
	(print s)
)


;; ========== testing code ========== 

;; the dodecahedron, a platonic solid, hard-coded by re-formatting an .obj file
(func dodecahedron
	(result (struct mesh))
	
	(let m (struct mesh) (alloc (struct mesh)))
	(set m vertices      (alloc (arr (vec 3 float))))
	(set m faces         (alloc (arr (vec 3 int))))
	(set m facenorms     (alloc (arr (vec 3 float))))
		
	(call add_vert m -0.436466 -0.668835 0.601794)
	(call add_vert m 0.918378 0.351401 -0.181931)
	(call add_vert m 0.886304 -0.351401 -0.301632)
	(call add_vert m -0.886304 0.351401 0.301632)
	(call add_vert m -0.918378 -0.351401 0.181931)
	(call add_vert m 0.132934 0.858018 0.496117)
	(call add_vert m -0.048964 0.981941 -0.182738)
	(call add_vert m 0.106555 0.162217 -0.980985)
	(call add_vert m -0.582772 0.162217 -0.796280)
	(call add_vert m -0.132934 -0.858018 -0.496117)
	(call add_vert m 0.048964 -0.981941 0.182738)
	(call add_vert m 0.582772 -0.162217 0.796280)
	(call add_vert m -0.106555 -0.162217 0.980985)
	(call add_vert m 0.436466 0.668835 -0.601794)
	(call add_vert m 0.730785 0.468323 0.496615)
	(call add_vert m -0.678888 0.668835 -0.302936)
	(call add_vert m -0.384570 0.468323 0.795474)
	(call add_vert m 0.384570 -0.468323 -0.795474)
	(call add_vert m 0.678888 -0.668835 0.302936)
	(call add_vert m -0.730785 -0.468323 -0.496615)

	(call add_face m 19  3  2)  (call add_face m 12  19  2) (call add_face m 15  12  2) 
	(call add_face m 8  14  2)  (call add_face m 18  8  2)  (call add_face m 3  18  2)
	(call add_face m 20  5  4)  (call add_face m 9  20  4)  (call add_face m 16  9  4)  
	(call add_face m 13  17  4) (call add_face m 1  13  4)  (call add_face m 5  1  4)
	(call add_face m 7  16  4)  (call add_face m 6  7  4)   (call add_face m 17  6  4)  
	(call add_face m 6  15  2)  (call add_face m 7  6  2)   (call add_face m 14  7  2)
	(call add_face m 10  18  3) (call add_face m 11  10  3) (call add_face m 19  11  3) 
	(call add_face m 11  1  5)  (call add_face m 10  11  5) (call add_face m 20  10  5)
	(call add_face m 20  9  8)  (call add_face m 10  20  8) (call add_face m 18  10  8) 
	(call add_face m 9  16  7)  (call add_face m 8  9  7)   (call add_face m 14  8  7)
	(call add_face m 12  15  6) (call add_face m 13  12  6) (call add_face m 17  13  6) 
	(call add_face m 13  1  11) (call add_face m 12  13  11)(call add_face m 19  12  11)

	(call calc_facenorms m)

	(return m)
)

;; test the ray caster
(func main (result int)
	(let m (struct mesh) (call dodecahedron))
	(call move_mesh m 0 0 5)
	
	(local light (vec 3 float) (alloc (vec 3 float) 0.1 0.2 0.4))
	(call render m light)

	(call destroy_mesh m)
	(return 0)
)


```
