How to deal with Project Sonar's data
From the beginning
30/01/2022
-----
## IDE
To start, you're going to want to be using an IDE - I'd reccommend [Visual Studio Code](https://code.visualstudio.com/). This guide is written assuming you're using VS Code, but everything will still work if you choose a different IDE.

Start by making a new folder to hold your project - I called mine ProjectSonarTutorial - and open it in VS Code. We're going to need to install Node.js too - a convenient way to do so is using a version manager like [nvm](https://github.com/nvm-sh/nvm) for Linux and MacOS, or [nvm-windows](https://github.com/coreybutler/nvm-windows) for Windows.

Once you have one of these installed (Note: On windows, you may have to restart to use nvm), we can install install Node.js. Open up a terminal as administrator (Or run the seccond command with `sudo` on linux/mac) and run `nvm install 16`. This will install the latest version of Node 16, currently 16.13.2 (We have to use Node 16 instead of the newer 17 as not all packages are currently compatible with it), then run `nvm use 16.13.2`. Now we have node.js installed and set up!

Returning to VS Code, we can open up its built in terminal with `ctrl + '`. We're going to need a few external packages later, so we might as well install them now. First up, we'll generate the package.json file (Where information like what packages your program depends on) is stored, by running `npm init`. `npm` stands for Node Package Manager, and is how you can install external packages (Like the MongoDB Node.js Driver) to use in your program. `npm init`'s defaults are good enough, however you can change them if you wish. Next up, open the `package.json` file that was created, and add the line `"type": "module",` below the description line - This marks our program as using the newer `import` syntax instead of the older `require` syntax. Be aware that some tutorials still make use of the old syntax, however. Finally, run `npm install mongodb` and `npm install tldts-experimental` to install the packages that we need.  (Explain generated package.json etc?)

Now we can begin to get to the interesting stuff: create a file called `index.js`. At the top of it, we can add
```
import { MongoClient } from "mongodb";
import { parse as tldParse } from "tldts-experimental";
import zlib from "zlib";
import fs from "fs";
import { get as getHttps } from "https";
import readline from "readline";
```
This imports what we need from the two packages we just installed, along with what we'll need from node's core modules.

We'll then add our main function:
```
/**
 * Main function
 */
async function main() {
	// Content of main function goes here
}


// Run the main function
main().catch(console.error);
```

To make sure everything is up and running, add the typical "Hello World" to the main function with `console.log("Hello World!");`. To run the program, go to the terminal and run `node index.js` - Hopefully you should be greeted with "Hello World!" being logged.
