HW-3 - TypeScript Refactor
==========================

Overview
--------

*   In this assignment, you will refactor your IGME-330 HW-2 Audio Visualizer into TypeScript
*   This will give you valuable experience working with TypeScript in a practical context

Requirements
------------

### Set up your project (10%)

> **⚠️ IMPORTANT - Test BEFORE You Convert!**
>
> Before you start the TypeScript conversion:
> 1. Run your HW-2 app in LiveServer and verify it works 100%
> 2. Post it to banjo and test that it works there too
>
> This ensures any issues you encounter are conversion-related, not pre-existing bugs!

*   Create a new folder named **lastName-firstInitial-hw3-refactor** (replace with your name)
*   Copy your completed HW-2 files into this folder
*   Ensure your HTML file is named **index.html**

###  Double check JavaScript (10%)

*   Convert all regular functions to arrow functions using `const` syntax
*   Ensure variable and function names follow course naming standards
*   Make sure code is organized into ES6 modules with proper `import` and `export`
*   ES6 classes (like your Sprite class) must be in their own distinct files

### Convert to TypeScript (40%)

*   Review: [Intro to TypeScript](https://github.com/rit-igm-web/igme-330-shared/blob/main/notes/intro-typescript.md "https://github.com/rit-igm-web/igme-330-shared/blob/main/notes/intro-typescript.md")

#### Initial Setup

Follow the instructions in [Intro to TypeScript - Use node & webpack](https://github.com/rit-igm-web/igme-330-shared/blob/main/notes/intro-typescript.md#iii-use-node--webpack-to-transpile-a-typescript-app-to-js) to set up your TypeScript tooling.

**Important setup notes:**

1. **Edit your HTML file** to use the transpiled code:
   - Comment out the `<script>` tag in the `<head>` of the document
   - Add `<script src="./dist/bundle.js"></script>` to the bottom of the page

2. **Update webpack.config.js** - Change `entry:` from **main.ts** to whatever your entry point file is

3. **Handle the "No inputs found" error:**
   - When you first run `npm run build`, you may see: `ERROR TS18003: No inputs were found in config file 'tsconfig.json'`
   - This happens because there are no `.ts` files yet 
   - **Don't** create an empty.ts file as a workaround - instead, move on to the next step

#### Convert Files to TypeScript

1. Change the file extension on **all** of your `.js` files to `.ts`

2. **Remove file extensions from imports** - this is critical!

```js
// CHANGE THIS:
import * as audio from './audio.ts';
import * as utils from './utils.ts';
import * as canvas from './canvas.ts';

// TO THIS (no file extensions):
import * as audio from './audio';
import * as utils from './utils';
import * as canvas from './canvas';

// DO THIS IN ALL OF YOUR FILES THAT HAVE IMPORTS
```

**Why?** TypeScript gives an error: `TS5097: An import path can only end with a '.ts' extension when 'allowImportingTsExtensions' is enabled.` The safest fix is to remove all file extensions from import statements.

3. Update **webpack.config.js** `entry:` to use the `.ts` extension

4. **Restart webpack** - Whenever you change a config file, press `Ctrl-C` to quit webpack, then run `npm run build` again

#### Fix TypeScript Errors

You'll now see TypeScript errors in your console. Work through them file by file.

##### **Pattern 1: Create types for Parameter Objects**

If you see something like: `Property 'showGradient' does not exist on type '{}'`

This means TypeScript doesn't know what properties your parameter object should have.

**Solution:** Create a type at the top of the file:

```ts
// In canvas.ts
interface DrawParams {
  showGradient: boolean,
  showBars: boolean,
  showCircles: boolean,
  showNoise: boolean,
  showInvert: boolean,
  showEmboss: boolean
}
// OR
type DrawParams = {
  showGradient: boolean,
  showBars: boolean,
  showCircles: boolean,
  showNoise: boolean,
  showInvert: boolean,
  showEmboss: boolean
}

// Then use it in your function:
const draw = (params: DrawParams) => {
  // ... your code
}
```

##### **Pattern 2: Type Assertions for DOM Elements**

If you see: `Property 'onclick' does not exist on type 'Element'`

This means TypeScript doesn't know what type of HTML element you're working with.

**Solution:** Use type assertions with `as`:

```ts
// CHANGE THIS:
const fsButton = document.querySelector("#btn-fullscreen");
const volumeSlider = document.querySelector("#volumeSlider");

// TO THIS:
const fsButton = document.querySelector("#btn-fullscreen") as HTMLButtonElement;
const volumeSlider = document.querySelector("#volumeSlider") as HTMLInputElement;
```

##### **Pattern 3: Type Event Targets in Event Handlers**

If you see: `Property 'value' does not exist on type 'EventTarget'`

```ts
volumeSlider.oninput = e => {
  audio.setVolume(e.target.value); // TypeScript doesn't know e.target's type
  // ...
}
```

**Solution:** Create a typed variable for the target:

```ts
volumeSlider.oninput = e => {
  const target = e.target as HTMLInputElement;
  audio.setVolume(target.value);
  // ...
}
```

##### **Pattern 4: String/Number Conversions**

If you see: `Type 'string' is not assignable to type 'number'` or `The left-hand side of an arithmetic operation must be of type 'any', 'number', 'bigint' or an enum type`

**Remember:** The `value` property of HTML `<input>` elements is ALWAYS a string!

**Solution:** Convert to the correct type:

```ts
// Convert to number when needed:
const numValue = Number(target.value);

// Convert to string when needed:
const strValue = String(someNumber);
```

##### **Pattern 5: Clean Up Legacy Audio Code**

If you see: `Property 'webkitAudioContext' does not exist on type 'Window & typeof globalThis'`

```ts
// CHANGE THIS:
const AudioContext = window.AudioContext || window.webkitAudioContext;

// TO THIS (we don't need webkitAudioContext anymore):
const AudioContext = window.AudioContext;

// Or just delete this line entirely and use window.AudioContext directly
```

##### **Pattern 6: Strongly Type Your Variables**

Add type annotations throughout your code:

```ts
// In audio.ts:
let audioCtx: AudioContext;
let element: HTMLAudioElement;
let analyserNode: AnalyserNode;
let gainNode: GainNode;

// In canvas.ts:
let ctx: CanvasRenderingContext2D;
let canvas: HTMLCanvasElement;

// In your main file:
const drawParams: DrawParams = {
  showGradient: true,
  showBars: true,
  // ...
};
```

#### Dealing with "Hints" vs "Errors"

TypeScript shows two types of issues:

*   **Errors** (red underlines) - Must be fixed for code to compile
*   **Hints** (gray dashed underlines) - Suggestions for better typing

**Strategy:**
1. Fix ALL errors first
2. Get the app working in the browser
3. THEN come back and address hints for cleaner code

#### Refine Your TypeScript

Once all errors are fixed and the app works, refine your TypeScript code:

**Create enums for constants:**
```ts
// src/enums/audio-defaults.enum.ts
export enum AudioDefaults {
  GAIN = 0.5,
  NUM_SAMPLES = 256
}
```

**Strongly type ALL the hints:**
- Go back and fix all those gray dashed underlines
- Add return types to functions
- Use union types where appropriate, e.g: `Type | OtherType | undefined`

#### Function Parameter Destructuring with TypeScript

Make your function signatures clearer with destructuring:

```ts
// Create an interface or type first
interface SpriteConfig {
  x: number;
  y: number;
  width: number;
  height: number;
}

// Then use destructuring in the constructor
constructor({ x, y, width, height }: SpriteConfig) {
  Object.assign(this, { x, y, width, height });
}

// Call it like this:
const sprite = new Sprite({ x: 0, y: 0, width: 100, height: 100 });
```

More info: [Destructuring function parameters with TypeScript](https://byby.dev/ts-object-destructuring)

#### Organize Your Project

Be sure all of your code is neatly organized into separate files and folders within a `src/` directory.

#### ES6 Classes with TypeScript

Properly type your ES6 classes:

```ts
// src/classes/Sprite.ts
export default class Sprite {
  x: number;
  y: number;
  width: number;
  height: number;

  constructor({ x, y, width, height }: SpriteConfig) {
    Object.assign(this, { x, y, width, height });
  }

  update(dt: number) {
    // ... if needed
  }

  draw(ctx: CanvasRenderingContext2D) {
    // ...
  }
}
```

### Common TypeScript Errors & Solutions

Here's a quick reference for errors you'll likely encounter:

| Error Message | What It Means | How to Fix |
|--------------|---------------|------------|
| `Property 'X' does not exist on type '{}'` | Missing type for parameter object | Create a type defining the properties |
| `Property 'onclick' does not exist on type 'Element'` | DOM element not typed | Use `as HTMLButtonElement` (or appropriate type) |
| `Property 'value' does not exist on type 'EventTarget'` | Event target not typed | Use `const target = e.target as HTMLInputElement` |
| `Type 'string' is not assignable to type 'number'` | Type mismatch | Convert with `Number(value)` or `String(value)` |
| `An import path can only end with a '.ts' extension` | Import has file extension | Remove `.ts` from import paths |
| `No inputs were found in config file` | No .ts files exist yet | Rename your .js files to .ts |

### Bundle the Code (15%)

*   Transpile the code to ES5 JavaScript and bundle it per the instructions in [Intro to TypeScript](https://github.com/rit-igm-web/igme-330-shared/blob/main/notes/intro-typescript.md#iii-use-node--webpack-to-transpile-a-typescript-app-to-js "https://github.com/rit-igm-web/igme-330-shared/blob/main/notes/intro-typescript.md#iii-use-node--webpack-to-transpile-a-typescript-app-to-js")
*   Bundle all code into a single `bundle.js` file in the **dist/** folder
*   Ensure your HTML file properly links to the bundled JavaScript: `<script src="./dist/bundle.js"></script>` at the bottom of the page

**Testing the bundled app:**
1. Check that **dist/bundle.js** contains ES5 code (look for `var` and `function` keywords)
2. Launch the app in LiveServer (you need a web server for Web Audio and Canvas)
3. Test all functionality - controls, audio, visualizations
4. Check the browser console for any runtime errors

### The app still works (15%)

*   The app should now:
    *   be written in modern TypeScript
    *   meet the class code style requirements
    *   have all of its code bundled into a single file - **bundle.js** that is linked to the **index.html** file
*   And it still functions as it should, both locally and on the banjo web server

### Documentation

*   (-10%) if not submitted
*   Create a file named **README.md** that includes:
    *   A brief description of how you implemented TypeScript in your project
    *   Any challenges you encountered during the conversion process
    *   Any improvements or optimizations you made to the code

### Submission

Submit to both:

*   The banjo web server (don't include node_modules or configuration files)
    *   Forget how to post to `banjo.rit.edu`? Get help here --> [https://github.com/tonethar/IGME-235-Shared/blob/master/notes/core-skills/ftp-upload-walkthrough.md](https://github.com/tonethar/IGME-235-Shared/blob/master/notes/core-skills/ftp-upload-walkthrough.md "https://github.com/tonethar/IGME-235-Shared/blob/master/notes/core-skills/ftp-upload-walkthrough.md")
    *   don't post unneeded files/folders to banjo (**package.json**, **node_modules**, TypeScript config files, etc)
    *   be sure to test the project on banjo and be 100% sure that it works
*   myCourses as a ZIP file
    *   1.  **README.md**

        2.  Your completed **lastName-firstInitial-hw3-refactor** folder (be sure to DELETE the **node_modules** folder!)

        *   Make sure to keep all the other tooling related files (**package.json**, TypeScript and webpack config files etc), just delete node_modules

        3.  The original version of your audio visualizer app:

        *   Put this in a folder named *lastName-firstInitial-original*
*   In the myCourses dropbox comments field, post the link to the project on banjo

---

Resources
---------

*   [Intro to TypeScript](https://github.com/rit-igm-web/igme-330-shared/blob/main/notes/intro-typescript.md "https://github.com/rit-igm-web/igme-330-shared/blob/main/notes/intro-typescript.md")
*   [HW-3 - Converting an existing project to TypeScript](https://github.com/rit-igm-web/igme-330-shared/blob/main/hw/hw3-typescript-notes.md "https://github.com/rit-igm-web/igme-330-shared/blob/main/hw/hw3-typescript-notes.md")
*   [TypeScript Handbook - Object types](https://www.typescriptlang.org/docs/handbook/2/objects.html)
*   [TypeScript Handbook - Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html)
