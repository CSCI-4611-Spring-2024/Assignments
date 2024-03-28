# Assignment 5: Artistic Rendering

**Due: Wednesday, April 10, 11:59pm CDT**

GLSL shaders make it possible for us to create some amazing lighting effects in real- time computer graphics. These range from photorealistic lighting to artistically inspired non-photorealistic rendering, as featured in games like *The Legend of Zelda: Breath of the Wild* and *Team Fortress 2*. In this assignment, you will implement GLSL shaders that can produce cartoon shading with a silhouette outline to complete the artistic effect.  You will also complete the implementation of a normal mapping shader that can render photorealistic textures on surfaces to make them appear more detailed and complex.

In this assignment, you will learn:

- How to calculate artistic per-pixel lighting in real-time.
- How to modify geometry on the fly to create viewpoint-dependent effects such as silhouette outlines.
- How to manipulate per-pixel normals to create the illusion of more complex surfaces.
- How to implement and use your own shader programs!

You can try out the [instructor's implementation](https://csci-4611-spring-2024.github.io/Assignments/Assignment-5/dist) in the Builds repository on the course GitHub.

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

## Useful Resources

he support code for this assignment will load several 3D model files and make it possible for you switch which one is being displayed, rotate it around with the mouse, and cycle between several different options for drawing the model with different shaders. Two shading modes (wireframe and unlit) are already implemented for you using built-in GopherGfx shaders. Your assignment is to implement four additional shading modes that take lighting into account in various ways. The first two of these will be the easiest because we will work on these in class.  Together, we will write a Gouraud shader that computes per-vertex lighting and a Phong shader that computes per-pixel lighting. The bulk of your assignment will be working on the second two shading modes, which build on these concepts to implement a cartoon-style artistic effect and normal mapping to produce detailed per-pixel variation in lighting on bumpy surfaces.

In addition to slides and examples from class, the [OpenGL 3.0 ES API Reference Card](https://www.khronos.org/files/opengles3-quick-reference-card.pdf) also provides a compact and useful resource for all the various functions provided by the shading language.  Note that the first three pages can be ignored, and the content relevant to GLSL is on pages 4-6.

## Program Requirements

For this assignment, you only need to modify the GLSL code in the `.vert` and `.frag` shader programs, which are located in `src/shaders`. However, as always, you are welcome to add to the TypeScript code if you want to add additional wizard functionality.

#### 1. Gouraud and Phong Shaders

We will work together in class to write a Gouraud shader and a Phong shader. Incorporating these into your code is worth one just point each because you can literally write them during class or even copy them from the implementations provided inthe GopherGfx library. However, **it is really important that you understand every line of code in these basic shaders before you try to implement more complex shaders.** So, please do not quickly copy and paste the code. Instead, go through the effort of actually typing each line of code into your solution. If you are struggling to understand how either these two shaders work, then you can make a post on Piazza or ask a TA to the walk you through the code during office or whiteboard hours.

#### 2. Artistic Cartoon Shading Using Texture Images

The strategy we will take to implement cartoon shading is quite flexible because rather than hard-coding the logic of the toon shading directly in the shader code (e.g., how many distinct colors to use for the cartoon effect), we will provide this data to the shader in the form of a "ramp" texture. With this approach, you can create hundreds of different shading effects with the same shader, just by swapping in/out different ramp textures!

Cartoon shading effects are widely used in games, as shown below.  *The Legend of Zelda: The Wind Waker* uses a very simplified light model. In this example, it looks like there are just two values used in the shading: each surface is either in bright light or dark. *Team Fortress 2* is a bit more subtle: it reduces the brightness variation in lit areas without completely flattening them out. You can read more about this in [Illustrative Rendering in Team Fortress 2](https://valvearchive.com/archive/Other%20Files/Publications/NPAR07_IllustrativeRenderingInTeamFortress2.pdf) by Mitchell et al.

![The Legend of Zelda: The Wind Waker](./images/zelda.jpg)

![Team Fortess 2](./images/tf2.jpg)

To implement cartoon shading in code, the vertex program can be the same as the Phong vertex shader described above. Much of the lighting calculation in the fragment shader will also be the same, but we will make a few tweaks.

Specifically, for the diffuse and specular components, our strategy will be to calculate the *light intensity* using the typical Phong model. However, instead of using these intensity values directly, we will use them as a lookup into a texture. Suppose that we use the dot product used to compute the diffuse lighting component, **n** &middot; **l**, to look up the texture.  Because this value represents the cosine of the angle between the two vectors, it will range from -1 to 1. We need to map this value to a texture coordinate, which will range from 0 to 1. 

If we use `standardDiffuse.png`, which is zero in the left half (corresponding to negative **n** &middot; **l**), and increases linearly from 0 to 1 in the right half (corresponding to positive **n** &middot; **l**), then we will get back the standard diffuse lighting term.  In other words, the object will have the same appearance as the Phong shader.

![standardDiffuse.png](./images/standardDiffuse.png)



However, if we use `toonDiffuse.png`, we will get something that looks like a cartoon, as if an artist were shading using just three colors of paint.

![toonDiffuse](./images/toonDiffuse.png)

It is worth noting that we could just hardcode these steps into the shader. For example, using an `if` statement to check the value of **n** &middot; **l** to see if it is between -1 and 0, 0 and 0.5, or 0.5 and 1.0, and then setting the output color to one of three predetermined values. However, using a texture as a lookup makes it really easy to create a wide variety of effects with a single shader just by switching the lookup texture. For example, we can easily add more bands of gray to the example image above, and it would look like the artist shaded the model using more markers. The starter code includes some extra ramp textures you can experiment with to try out different effects, and you can certainly try creating your own textures as well. Writing the shader this way is also a great learning tool because many advanced shaders rely upon this strategy of using a texture to provide more abstract "input data" to achieve various visual effects.

In terms of the actual implementation, you should write your shader in the `toon.vert` and `toon.frag` files.  As a first step, you should copy the contents of your Phong shaders into the `main()` function. You can then isolate the math that calculates the *diffuse light intensity* and *specular light intensity* and use these values for your texture lookups.

For the diffuse component, the *diffuse light intensity* to use for the texture lookup will be **n** · **l**.  Because this value represents the cosine of the angle between the two vectors, it will range from -1 to 1, which is not quite the correct range for *uv* texture coordinates. So, you will need to convert this to the correct texture coordinates. Remember, you want your texture coordinates to work so that if the value of **n** · **l** is −1, then you use the color on the leftmost side of the texture. If it is 1, then you use the color on the rightmost side of the texture. 

Also, note that the color in these ramp textures only varies from left to right, so the *v* texture coordinate (the up-down coordinate) really does not matter. You could use 0.0 0.5, 1.0 -- any value for *v* should produce the same results. The key is to calculate the correct value for the *u* texture coordinate. Use the GLSL built-in function `texture()` to get the color from the ramp image at the coordinates you calculate.

For the specular component, you should use the same strategy with just a couple of differences. First, remember, the *specular light intensity* is calculated using a slightly different equation. Also, this equation will typically include something like max(0, *specularIntensity*), effectively trimming the range of the function to 0 to 1. The specular ramp textures are designed to match this range, with 0 intensity on the left of the image and +1 intensity on the right of the image. So, you can use this 0...1 value directly when doing your texture lookup.

When you get to this point in your implementation of the assignment, you should check your result against the following reference images (and the instructors' implementation). With the sphere mesh selected, the rendering should look like similar to the following as you progress through the assignment:

![reference images](./images/reference.png)

1. The sphere rendered with the Unlit shader provided by GopherGfx.
2. The sphere with Gouraud shading.
3. The sphere with Phong shading. Also the same as cartoon shading using the `standardDiffuse.png` and `standardSpecular.png` ramps. To test this, you can switch the texture ramps loaded by the `ToonMaterial` during program initialization.
4. Cartoon shading with `toonDiffuse.png` and `toonSpecular.png`.
5. Cartoon shading with a silhouette outline of thickness 0.01 (described in the next section).

## Rubric

Graded out of 20 points.  Partial credit is possible for each step.

## Wizard Bonus Challenge

To be added.

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

## License

Material for [CSCI 4611 Spring 2024](https://github.com/CSCI-4611-Spring-2024/Syllabus) by [Evan Suma Rosenberg](https://illusioneering.umn.edu/) is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/).  Please do not distribute outside the course without permission from the instructor.