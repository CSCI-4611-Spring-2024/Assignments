# Assignment 4: So You Think Ants Can Dance

**Due: Wednesday, March 27, 11:59pm CDT**

Animated characters are an important part of computer games and other interactive graphics. In this assignment, you will learn how to animate computer graphics characters using data from motion capture systems. You will be working with data from the [Carnegie Mellon motion capture database](http://mocap.cs.cmu.edu/), a great resource of free "mocap" data. The data will be formatted as text files, but with your programming and math skills, you will bring this data to life on the dance floor!

The CMU motion capture data is typical of all the skeleton-based motion capture data found in games and movies. So, gaining some experience with this type of animation is one of the most important goals of the assignment. There are several important learning goals, including:

In completing this assignment, your goal should be to learn:

- How transformations can be composed in a hierarchy to create a scene graph and/or, in this case, an animated character within a scene.
- How transformations can be used to scale, rotate, and translate basic shapes (unit cubes, spheres, cylinders, and cones) into more complex scenes and characters.
- How mocap data can be used and manipulated in multiple ways to create different types of animations. For example:
  - How to create a looping animation that smoothly interpolates between the beginning of the motion clip and the end to avoid any discontinuities.
  - How to overlay new motion clips onto a character at runtime, for example, making your character jump in a game when you press a button, or in our case, perform one of a series of cool ballet moves.
- How to read and extend some fairly sophisticated computer graphics code.

The amount of code you will need to write to complete this assignment is less than in the previous assignments. The support code provides quite a bit of infrastructure to deal with reading and playing back the mocap data. So, the key challenges will come in reading through and understanding the existing code as well as really thinking about the code that you do write. You will probably need to work out some of the structure on paper before sitting down at the keyboard to program.

You can try out the [instructor's implementation](https://csci-4611-spring-2024.github.io/Assignments/Assignment-4/dist) in the Builds repository on the course GitHub.

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

## Data

The CMU Mocap database contains 2,605 different motions, most recorded at 120Hz, but some recorded at 60Hz or other speeds. These motions range from the simple (person walking straight forward), to the complicated (directing traffic), to the silly (someone doing the "I'm a little teapot" dance).

The motions in the CMU database use skeletons specified in .asf files and separate motions specified in .amc files. The .asf files specify bone names, directions, lengths, and the skeleton hierarchy for one specific human subject who came to the mocap lab. That person likely performed several motions during the capture session, so there is typically one .asf file for multiple .amc files. The subjects are numbered (e.g., subject #50). and the skeleton files are named accordingly (e.g., 50.asf). Motion filenames start with the subject ID, then have an underscore, then the number of the motion (e.g., 50_01.amc is the first motion captured for subject 50). The support code comes with the data files we used in our solution to the assignment, but it can be fun to swap in other motions, and you are encouraged to experiment with this by downloading other .asf and corresponding .amc files from the [CMU database](http://mocap.cs.cmu.edu/).   If you are interested, you can also read more about the [Acclaim Motion Capture](http://graphics.cs.cmu.edu/nsp/course/cs229/info/ACCLAIMdef.html) data format used by the mocap files. 

The animations used in this assignment are based on raw mocap data that contains noticeable glitches.  For example, you may notice the ballet dancer's limbs sometimes rapidly jump back and forth between incorrect positions, which also occurs in the instructor's implementation. These visual glitches often occur when the cameras could only capture a partial view as the person moves around. In a production pipeline, the raw mocap data would be manually cleaned by an animator before being integrated in an application. You **do not** need to worry about the mocap glitches during this assignment.

## Program Requirements

Your assignment is to extend the support code to complete two dancing ants "scenes". In the Ballet Studio scene, we should see a single animated character that responds dynamically to the buttons in the GUI. This is a simplified version of what you could expect in a game where a character is controlled dynamically by a gamepad. The character should repeat a looping "idle" animation, waiting for the user to control them in some way. Then, the character should respond when we press the "Play Motion 1", "Play Motion 2", etc. buttons in the GUI. (You can think of pressing these buttons as equivalent to pressing the "A", "B", or "D-pad" buttons on a game controller.) The character should smoothly transition from the idle motion to the new motion, and when the new motion is done playing, they should transition back to the idle motion. In this way, the Ballet Scene is intended to illustrate how to work with mocap data in a dynamic context. The Ballet Scene is also really useful for debugging because the GUI includes a dropdown list you can use to change the character shown in the Ballet Scene. You are welcome to add more characters, but you need to implement at least the four different characters that you see already listed in the dropdown. The two "Axes" characters and the "Skeleton" character should be useful for debugging your code. For the "Ant" character, you are welcome to create an ant, like the instructors' implementation, or you can make your own custom character (subject to the specific requirements in the rubric below).

The next scene is the "Salsa Class". This scene should show two ant characters (or two of you own custom characters) dancing together. This animation is not dynamically controlled. So, in a sense, it is a simpler situation than in the Ballet Studio, but it should look quite convincing to see the multiple characters dancing together.

For this assignment, we want to emphasize that **you do need to read the support code**. Reading through all of the code provided to learn the structure of the program is part of this assignment. Try to understand how the skeletal data are stored and how the motion data are stored in Keyframes. Understand what happens when you ask the character's AnimationController to play an Animation and to overlay a Animation. Understand how custom character geometry can be created by adding meshes as children of specific Bones in the Skeleton.

Once you understand these aspects of the support code and animation system, we suggest proceeding with your implementation by following along in the described in the rubric below.

## Rubric

Graded out of 20 points.  Partial credit is possible for each step.

**Part 1: AxesCharacterGeometry.ts** (5 points total)

- Complete the code in the `createGeometryRecursive()` method to draw the coordinate axes in the correct position for each bone. (1 point)
- Call the method recursively for each of the bone's children to propagate through the entire skeleton. (2 point)
- Set the rotation of the coordinate axes to look in the direction of the bone. If this is implemented correctly, then the blue line (+Z axis) should be pointing towards the parent of each bone, as shown in the instructor's implementation. (2 point)

**Part 2: SkeletonCharacterGeometry.ts** (6 points total)

- Complete the code in the `createGeometryRecursive()` method to draw a skeleton mesh (cylinder) for each bone. (1 point)
- Then, scale, rotate, and translate the cylinder so that it starts at the origin of the bone and extends in the bone's direction with a length equal to the bone's length. (3 points, 1 for each transformation)
- Lastly, compose the matrices in the correct order and set the localToParentMatrix. (2 points)

If this part is correct, a stick figure should show up when you choose the Skeleton character, as shown in the instructor solution. There should not be gaps between bones, although you will see the end caps of the cylinder when the joint bends a lot.

**Part 3: AntCharacter.ts** (6 points total)

- **Part 3.1:** Create a custom humanoid character (it can be an ant like the example solution or some other character you design yourself) by adding geometries for specific bones in the skeleton. If the geometries are big, as in the ant in the instructor's solution, then you may not need to define a geometry for *every* bone; do what looks good for your character, but and at a minimum, define 4 geometries that are uniquely translated, rotated, and scaled to create a convincing custom character. (3 points)

- **Part 3.2:** Create a face for your character under the "‘head" bone. Note, there are no "bone" placeholder for the eyes and other facial features, so this is a chance to show us you know how to compose matrices together to create a more sophisticated 3D model. Your character's face should have at least 3 unique geometries and should demonstrate your knowledge of composing transformations; at least one part of the face should adjust the geometry’s position, the rotation, *and* the scale, like the antennae on the instructor solution. (3 points)

*Note that although the class is named AntCharacter, you are not required to required to create an ant.  If you have another creative idea, you may want to rename this class to better describe your implementation.*

**Part 4: App.ts**. (3 points total)

- **Part 6.1:** Load 4 additional animations from the CMU motion database. (1 point)

- **Part 6.2:** Trim each new animation so there is no “idle” time at the beginning and end of each clip. (1 point)

- **Part 6.3:** Add each animation to the ballet character's overlay queue when the appropriate GUI button is clicked. (1 point)

Please pick your motions such that the character does not end up off screen when you run through the animations in sequence 1, 2, 3, 4, 5.  This will make it more efficient for the TAs to grade.

## Wizard Bonus Challenge

All of the assignments in the course will include great opportunities for students to go beyond the requirements of the assignment and do cool extra work. On each assignment, you can earn **one bonus point** for implementing a meaningful new feature to your program. This should involve some original new programming, and should not just be something that can be quickly implemented by copying and slightly modifying existing code.  

There are great opportunities for extra work in this assignment. For example:

- You could browse through the [CMU mocap database](http://mocap.cs.cmu.edu/), download more skeleton (.asf) and motion clip (.amc) files, and then add additional characters to the scene.
- You might consider extending one of the characters with mouse and/or keyboard input.  For example, you could make a character that is animated using mocap data, but moves in the direction controlled by the user.
- You could create or download some 2D sprite animations and add them to the 3D scene, as discussed in class.
- You could extend the animation system with your own unique idea. Creativity is encouraged!

If you do any wizardly work, please describe it in your `README.md` file so we know what to look for!

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

- Assignment developed by Daniel Keefe and TAs 2012, 2014.

- Ported to MinGfx and additional refinements by Daniel Keefe and TAs 2018-2021.

- Ported to GopherGfx and additional refinements by Evan Suma Rosenberg and TAs 2022-2023.
- Significant revisions to emphasize use of transformation matrices by Daniel Keefe 2023.
- Significant revisions and transition to GopherGfx animation system by Evan Suma Rosenberg 2024.

## License

Material for [CSCI 4611 Spring 2024](https://github.com/CSCI-4611-Spring-2024/Syllabus) by [Evan Suma Rosenberg](https://illusioneering.umn.edu/) is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/).  Please do not distribute outside the course without permission from the instructor.