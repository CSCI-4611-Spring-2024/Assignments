# Assignment 6: A World of Drawings

**Due: Monday, April 29, 11:59pm CDT**

The culminating assignment in the course is inspired by the 1955 children's book, [Harold and the Purple Crayon](https://en.wikipedia.org/wiki/Harold_and_the_Purple_Crayon) by Crockett Johnson. Harold is a little boy who creates his own virtual worlds just by drawing them with his magical purple crayon. In this assignment, you will bring Harold's magic to your computer screen.

In addition to inspiring children for decades, Harold can also claim to have inspired some exciting early computer graphics research in non-photorealistic rendering. In this assignment, we will be implementing portions of the paper, [Harold: A World Made of Drawings](https://dl.acm.org/citation.cfm?id=340927), presented by Cohen et al. at ACM NPAR 2000, the 1st International Symposium on Non-Photorealistic Animation and Rendering. In addition to reading the paper, you can also watch a [video of the original system](https://mediaspace.umn.edu/media/t/1_gtj35asj). 

The lead author of this paper, Jonathan Cohen, was about your age (a junior or senior in college) when he developed this system and published it as his first research paper. He then went on to work on movie special effects at Industrial Light and Magic. This is the kind of thing you may also be able to do if you continue studying computer graphics and get involved in the graphics, visualization, and virtual reality research groups in our department!

In this assignment, you will learn:

- How to perform 3D mesh editing operations in response to user input.
- How to use pick rays and intersection tests to convert 2D screen space coordinates (e.g., the mouse position, a 2D curve onto the screen) into a 3D virtual world.
- How to implement a computer graphics algorithm described in a research paper.

This is a really playful assignment, and we hope you have fun working on it. Please feel free to save snapshots or videos of any awesome worlds you create and share them on the class Piazza forum!

You can try out the [instructor's implementation](https://csci-4611-spring-2024.github.io/Assignments/Assignment-6/dist) in the Builds repository on the course GitHub.

## Repository Setup

We are using GitHub classroom for submission of programming assignments.  When you accept the first assignment, you will need to select your x500 from the class roster. The system will then create a new private repository with template code that is only accessible by you, the instructor, and the TAs. You will need to create a GitHub.com account if you do not already have one.  Note that this is different from the University's github.umn.edu account.  

**Step 1:** Create your private repository using the following link: TO BE ADDED

**Step 2:** Your repository will be added to the [GitHub course organization](https://github.com/CSCI-4611-Spring-2024).  You will need to open your repository on GitHub, then go to Settings->Pages and change the build and deployment source to `GitHub Actions`, as shown in the image below.  You **must** complete this step or you will not be able to to deploy the finished assignment.

**Step 3:** Check out the code to your local machine **using a git client**. If you are not familiar with using `git` from the command line, then I recommend using a GUI-based client such as [GitHub Desktop](https://desktop.github.com/), [GitKraken](https://www.gitkraken.com/), or the [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) extension for VS Code. 

Do **not** simply download the contents of your repository as a ZIP file!  This is not the right way to use version control, and you will be unable to push your local changes back to the GitHub repository. 



![GitHub screenshot](./images/github.jpg)



## Prerequisites

To work with this code, you will first need to install [Node.js 20.11.0 LTS](https://nodejs.org/en/) (or newer) and [Visual Studio Code](https://code.visualstudio.com/). 

I also recommend you install the following useful VS Code extensions:

- [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) (makes source control easier)
- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint) (static code analysis tool that can flag errors)
- [JavaScript Debugger](https://marketplace.visualstudio.com/items?itemName=ms-vscode.js-debug-nightly) (essential for real-time debugging)
- [WebGL GLSL Editor](https://marketplace.visualstudio.com/items?itemName=raczzalan.webgl-glsl-editor) (used for programming shaders later in the course)

## Getting Started

The starter code implements the general structure that we reviewed in lecture.  After cloning your repository, you will need to set up the initial project by pulling the dependencies from the node package manager with:

```
npm install
```

This will create a `node_modules` folder in your directory and download all the dependencies needed to run the project.  Note that this folder is listed in the `.gitignore` file and should not be committed to your repository.  After that, you can compile and run a server with:

```
npm run start
```

Your program should open in a web browser automatically.  If not, you can run it by pointing your browser at `http://localhost:8080`.

## Program Overview

This is a fairly complete graphics application.  You should start by reading through the code, which is heavily commented, and learn how a computer graphics program like this is put together.

The `DrawingApp` class is the main application class and implements the sketch-based user interaction techniques. One of the things that is really cool about this application is that the user input can be interpreted differently based upon the current content. The original Harold paper did include some drawing modes in order to support some extra features, but for our version, we do not need any buttons to turn on "draw on the sky mode" vs. "draw on the ground mode" vs. "create a billboard mode" – we just draw, and the system figures out our intent. Properties of the path drawn by the user's mouse (called the *stroke*) are used to trigger different 3D modeling operations, as shown in the following table:

| Stroke Made by Mouse                     | 3D Modeling Operation                            |
| ---------------------------------------- | ------------------------------------------------ |
| Starts in the sky                        | Add a new stroke to the sky                      |
| Starts AND ends on the ground            | Edit the ground mesh to create hills and valleys |
| Starts on the ground and ends in the sky | Create a new billboard                           |
| Starts on an existing billboard          | Add the stroke to the existing billboard         |

As the user draws a stroke, the application records the path of the mouse as a series of 2D points, which are stored in the `Stroke2D`  object called `currentStroke`. The Stoke2D object has two jobs. First, it stores the path drawn by the user's mouse as a series of 2D points which you can access with `currentStroke.path`. Second, it creates a triangle mesh so we can draw the stroke on the screen. You can access the vertices and indices of this mesh with `currentStroke.vertices` and `currentStroke.indices`. All of the data stored in  `currentStroke` is 2D,  which means that the points are defined in **normalized device coordinates**.  Remember, in this coordinate space, the top right corner of the screen is (1, 1) and the bottom left is (-1, -1).  

After the user is finished drawing the 2D stroke, the program then must decide how to handle it. This is where the assignment code begins. You will be implementing four key features of the system:

1. Drawing strokes in the sky.
2. Drawing hills and valleys on the ground.
3. Drawing billboards, which allow the user to create strokes that show on top of the ground.
4. Walking around and enjoying the scenery.

## Part 1: Drawing Strokes in the Sky

In the `App` class, the support code already determines for you whether the stroke should be added to the sky or used to create a new billboard or to edit the ground. Please look at the code in the `onMouseDown()` and `onMouseUp()` functions to see how this works. It is a really interesting approach to a sketch-based user interface!

When it determines that the user is drawing a "sky stroke", it will call the `createSkyStrokeMesh()` method in the `Stroke2D` class. You will need to write the code to complete this function. The basic strategy is to use the 2D stroke data to create a new `Mesh3` object that has the same triangles as in the original stroke, but with the vertices projected onto a large sphere that represents the "sky."

## Part 2: Drawing Billboards Attached to the Ground

Part 2 of the assignment is similar to drawing strokes in the sky, but this time you need to complete the `createBillboard()` method in the `Stroke2D` class.  This function handles situations in which a user draws a stroke that starts on the ground and ends in the air or draws a stroke that starts on an existing billboard. In these cases, we need to define a 3D plane that is "anchored" to the ground at the point where the user began their stroke and project the stroke onto this plane. Like drawing in the sky, this requires creating a new `Mesh3` object that has the same triangles as the original 2D stroke, but with the vertices moved to new locations.

Like traditional billboards in computer graphics, we will want this new mesh to rotate to face the camera, and the program is already setup to do this for you. You just need to create a new `Billboard` object, which provides a small wrapper around a `Mesh3 `that adds the capability to rotate and face the camera.

## Part 3: Drawing Hills and Valleys on the Ground

This part of the assignment will require completing the `reshapeGround()` method in the `Ground` class. This is the most complicated part of the assignment. Rather than creating a new `Mesh3` based on the stroke that is drawn, this feature involves modifying the existing 3D ground mesh.

The vertices of the ground mesh have already been created. It is a regular grid of squares, each of which is divided into two triangles, similar to the planar Earth mesh in our previous assignment. To simplify the mesh-editing algorithm, we will not do any adding or subtracting of vertices. All you need to do is move the vertices' positions up or down to create hills and valleys. Note that this will mean that some of the triangles could end up being fairly stretched out, and you will probably notice some lighting artifacts and geometric distortions. That is OK, we can live with a few artifacts given that we are drawing with crayons anyway! Also, it isn't too distracting since we are using toon/outline shaders. 

The specification for how to edit the ground in response to the stroke drawn by the user comes from the Harold research paper. We will follow the algorithm and equations described in Section 4.5 of the paper, which is quoted here:

---

Terrain-editing strokes must start and end on the ground. Call the starting and ending points *S* and *E*… [T]hese two points, together with the **y**-vector, determine a plane in *R3*, that we call the *projection plane*. The points of the terrain-editing stroke are projected onto this plane (this projection, which is a curve in *R3*, is called the *silhouette curve*); the shadow of the resulting curve (as cast by a sun directly overhead) is a path on the ground (we call this the *shadow*). Points near the shadow have their elevation altered by a rule: each point *P* near the shadow computes its new height (*y*-value), *P'y* , as a convex combination

![](C:\Teaching\CSCI-4611-2023-Spring-Prep\Assignments (Finished)\Assignment-6\images\equation1.png)

*(This equation is edited slightly from the original text to include the two cases.)*

where *d* is the distance from *P* to the projection plane, *h* is the *y*-value of the silhouette curve over the nearest point on the projection plane to *P*, and *w*(*d*) is a weighting function given by

![](C:\Teaching\CSCI-4611-2023-Spring-Prep\Assignments (Finished)\Assignment-6\images\equation2.png)

This gives a parabolic cross-section of width 10 for a curve drawn over level terrain. Other choices for *w* would yield hills with different shapes that might be more intuitive, but this particular choice gives reasonable results in most cases.

Note that if the silhouette curve bends back on itself (i.e. it defines a silhouette that cannot be modeled using a heightfield), then the variation of height along the shadow will be discontinuous. The resulting terrain then may have unexpected features.

---

The *h* in the equation is a complex to calculate, so we have already implemented a function called `computeH()` that you can use to calculate *h* given the silhouette curve, the projection plane, and the closest point on the projection plane to the vertex we are editing.

## Part 4: Walking Around

For the final part of this assignment, you will add some code to the `update()` method of the `App` class.  Currently, the user can move the camera using the WASD keys,  and the height always remains at a fixed value.  This means that if the user draws a hill, then they could end up walking into and even underneath the ground mesh.  Instead, we want to adjust the camera height dynamically based on the elevation of the ground.

To adjust the camera height, you will need to perform a ray cast directly downwards based on the camera's position in X and Z, figure out where that ray intersects the ground mesh, and then adjust the Y position of the camera.

One potential way to accomplish this would be to perform an intersection test against all the triangles in the `Ground` mesh.  However, this would be very inefficient.  The mesh is a 200 x 200 grid, and similar to the Earth mesh from Assignment 3, each rectangular cell contains 2 triangles.  This means that under the hood, performing a ray cast directly against this mesh would require 80,000 ray-triangle intersection tests!

Fortunately, we have a much faster solution because the grid mesh has a consistent topology, and the XZ coordinates of each vertex remain consistent. The `Ground` class already contains a method called `getTriangleAtPosition()`, which returns the three vertices that make up the triangle directly underneath the camera's XZ position. You can perform a single ray-triangle intersection test using the `intersectsTriangle()` method and adjust the camera height as described above.  This approach allows us to skip intersection tests against the other 79,999 triangles in the mesh, which would drastically slow down the program!

## Rubric

Graded out of 20 points.  Partial credit is possible for each step.

**Part 1: Drawing in the Sky** (6 points total)

- Create a bounding sphere of the appropriate radius. (1)
- Create at least one pick ray originating at the camera. (1)
- Create and return a new `Mesh3` from the `createSkyStrokeMesh()` method. (1)
- Correctly set the vertices of the `Mesh3` to project the input stroke onto the sky. (3)

**Part 2: Drawing Billboards Attached to the Ground** (4 points total)

- Define the correct billboard plane in the `createBillboard()` method. (2)
- Return a new `Billboard` object with mesh, normal, and anchor point set correctly (2)

**Part 3: Drawing Hills and Valleys on the Ground** (7 points total)

- Correctly define the projection plane for the stroke. (3)
- Correctly update the vertices of the ground mesh. (4)

**Part 4: Walking Around** (3 points total)

- Cast a ray downwards from the camera to find the intersection point with the ground triangle. (2)
- Adjust the height of the camera to be the correct distance above the ground mesh. (1)

**Submission**

- To complete the submission, you will need to update your `README.md` file, build your project, and then commit/push both your code **and** the contents of the `dist` folder to GitHub as described below.  (-2 point deduction for skipping these steps)

## Wizard Bonus Challenge

All of the assignments in the course will include great opportunities for students to go beyond the requirements of the assignment and do cool extra work. On each assignment, you can earn **one bonus point** for implementing a meaningful new feature to your program. This should involve some original new programming, and should not just be something that can be quickly implemented by copying and slightly modifying existing code.

There are great opportunities for extra work in this assignment. There are a few issues you will discover if you use the program long enough; for example, if you attach a billboard to the ground and later modify the ground, the billboard will not move up/down with the new height of its anchor point. This would be a great feature to add. 

Also, if you go back to the original paper and video, you will notice a number of other exciting features that are part of the original system, such as *bridge strokes*, *ground strokes*, or *navigation strokes*. Alternatively, you could write some different stroke shaders to implement watercolor brush strokes.  It would also be cool to see these drawing somehow come alive!  For example, you could introduce some of the concepts you have already learned about animation.

As always, completely original and creative ideas are encouraged!

## Building and Deploying to GitHub Pages

When you have finished the assignment, you should complete the missing information in your `README.md` file. Then, will need to run the following command to generate a build:

```
npm run build
```

This compiles your TypeScript program into a JavaScript bundle that will be placed in the `dist` folder. To complete your submission, you should commit your code changes **and** the contents of the `dist` folder to your repository, and then push to GitHub. This will trigger a server-side workflow that will automatically deploy your build as a website on GitHub pages. Note that you may need to wait a minute or two for the deployment to become active.

Make sure to test everything by pointing your web browser at the GitHub pages URL for your repository:

```
https://csci-4611-spring-2024.github.io/your-repo-name-here
```

If your program runs correctly, then you are finished! Note that the published JavaScript bundle code generated by the TypeScript compiler has been obfuscated so that it is not human-readable. So, you can feel free to send this link to other students, friends, and family to show off your work!

## Acknowledgments

- Original assignment developed by Daniel Keefe and TAs, 2012, 2014.

- Significant additions, including silhouette rendering, by Rahul Narain and TAs, 2016.

- Ported to MinGfx and additional refinements by Daniel Keefe and TAs, 2018-2021.
- GopherGfx implementation and significant additions, including normal mapping and stencil buffer silhouette rendering, by Evan Suma Rosenberg, 2022-2023.
- Significant revisions to assignment description by Daniel Keefe and TAs, 2023.
- Additional revisions to assignment description by Evan Suma Rosenberg, 2024.

- The PBR textures and normal maps were obtained from [3dtextures.me](https://3dtextures.me/).

## License

Material for [CSCI 4611 Spring 2024](https://github.com/CSCI-4611-Spring-2024/Syllabus) by [Evan Suma Rosenberg](https://illusioneering.umn.edu/) is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/).  Please do not distribute outside the course without permission from the instructor.
