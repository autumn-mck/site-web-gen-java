How to deal with Project Sonar's data
From the beginning
01/02/2022
nodejs, project sonar, programming
-----
## What is Project Sonar?
[Project Sonar](https://opendata.rapid7.com/) is a data collection project containing information from scans across the internet: [DNS records](https://en.wikipedia.org/wiki/Domain_Name_System), SSL Certificates, and also scans many commonly used ports for TCP/UDP.

### Why should you use it?
Project Sonar's forward DNS data can be used as a reverse DNS lookup (Finding a list of domains that point to a given IP address) more reliably than the standard method ([PTR Records](https://www.cloudflare.com/learning/dns/dns-records/dns-ptr-record/)).

It can also be used for subdomain enumeration (Finding subdomains under the given domain), which can reveal web applications and other services are publicly exposed to the Internet.

The port scans can also be used to guess at what software is running and publicly exposed.


## Setup
To start, you're going to want to be using an IDE - I'd reccommend [Visual Studio Code](https://code.visualstudio.com/). This guide is written assuming you're using VS Code, but everything will still work if you choose a different IDE. It's also assuming you've not used Node.js before - if you have, you might want to skip to [Start Programming](#start-programming)

Start by making a new folder to hold your project - I called mine ProjectSonarTutorial - and open it in VS Code. We're going to need to install Node.js too - a convenient way to do so is using a version manager like [nvm](https://github.com/nvm-sh/nvm) for Linux and MacOS, or [nvm-windows](https://github.com/coreybutler/nvm-windows) for Windows.

Once you have one of these installed (Note: On windows, you may have to restart to use nvm), we can install install Node.js. Open up a terminal as administrator (Or run the seccond command with `sudo` on linux/mac) and run `nvm install 16`. This will install the latest version of Node 16, currently 16.13.2 (We have to use Node 16 instead of the newer 17 as not all packages are currently compatible with it), then run `nvm use 16.13.2`. Now we have node.js installed and set up!

We're also going to be using MongoDB - I used a local installation for this tutorial. To install it, follow the instructions over at <https://docs.mongodb.com/manual/installation/> MongoDB compass might be installed along side it, but if not, I'd reccommend installing it too.

Returning to VS Code, we can open up its built in terminal with `ctrl + '`. We're going to need a few external packages later, so we might as well install them now. First up, we'll generate the package.json file (Where information like what packages your program depends on) is stored, by running `npm init`. `npm` stands for Node Package Manager, and is how you can install external packages (Like the MongoDB Node.js Driver) to use in your program. `npm init`'s defaults are good enough, however you can change them if you wish. Next up, open the `package.json` file that was created, and add the line `"type": "module",` below the description line - This marks our program as using the newer `import` syntax instead of the older `require` syntax. Be aware that some tutorials still make use of the old syntax, however. Finally, run `npm install mongodb` and `npm install tldts-experimental` to install the packages that we need.  (Explain generated package.json etc?)

## Start programming

Now we can begin to get to the interesting stuff: create a file called `fetchData.js`. At the top of it, we can add:
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

To make sure everything is up and running, add the typical "Hello World" to the main function with `console.log("Hello World!");`. To run the program, go to the terminal and run `node fetchData.js` - Hopefully you should be greeted with "Hello World!" being logged.

Next up we'll connect to MongoDB. Since we're using a local database, the connection URI should be as simple as `"mongodb://localhost:27017"`. Then we can create a new MongoClient, and pass our connection string to its constructor. Then we can open the connection with `await client.connect();` To make sure everything is working, we can print a list of all databases. Let's make a function for it!

```
async function listDatabases(client) {
    let dbList = await client.db().admin().listDatabases();
 
    console.log("Databases:");
    dbList.databases.forEach(db => console.log(` - ${db.name}`));
};
```

Call the new listDatabases function from within our main function, and pass it the MongoClient we created, after opening its collection. Running our code so far (with `node fetchData.js`) we should get something like this:
![List of databases](ListDatabases.png)

Your code so far should be similar to
```
import { MongoClient } from "mongodb";
import { parse as tldParse } from "tldts-experimental";
import zlib from "zlib";
import fs from "fs";
import { get as getHttps } from "https";
import readline from "readline";

/**
 * Main function
 */
async function main() {
	// Database is currently hosted on same machine
	const uri = "mongodb://localhost:27017";
	const client = new MongoClient(uri);

	try {
		// Connect to MongoDB
		await client.connect();
		
		// List databases
		await listDatabases(client);
	} catch (e) {
		console.error(e);
	}
}

async function listDatabases(client) {
	let dbList = await client.db().admin().listDatabases();

	console.log("Databases:");
	dbList.databases.forEach((db) => console.log(` - ${db.name}`));
}

// Run the main function
main().catch(console.error);
```

You might have have noticed that the program still appears to be running, and you can no longer type in the terminal. You can press Ctrl + c when focused on the terminal to stop the currently running program at any time.

2 Options:

## Fetching a local copy of Project Sonar
Project Sonar's data can be found at <https://opendata.rapid7.com/sonar.fdns_v2/>. I'll be using the DNS 'A' records for this guide, but I'll be talking more about what the other items are later. Since we're using the A records, we need the file ending in `-fdns_a.json.gz`. Do note that the file is large (17gb) and be careful not to unzip it - uncompressed, it is over 200gb!

Let's add a new function, `readFromFile`.

```
async function readFromFile(client) {
	const sonarDataLocation = "fdns_a.json.gz";
	let stream = fs.createReadStream(sonarDataLocation);
	parseSonar(client, stream);
}
```

`sonarDataLocation` should be wherever you saved the data to - either a relative path, in the current case, or an absolute path, like `C:\\Users\\James\\Downloads\\fdns_a.json.gz`. We then create a read stream - not the actual data itself - that we can later read through and parse. `fs` is Node.js's filesystem module, allowing us to interact with local files. We then pass this stream, and the MongoClient passed into the function, to a function that does not yet exist - it's next to do.

Finally, let's call this method from the main function with `readFromFile(client);`

Alternatively:

## Fetching an online version of Project Sonar
This method is a bit more complicated, but means that we do not have to keep a copy saved on our machine, taking up space. It will require you to have a reliable internet connection, however.

Let's add a new function, `readFromWeb`.
```
async function readFromWeb(client, url) {
	getHttps(url, function (res) {
		// Code here
	}).on("error", function (e) {
		console.error(e);
	});
}
```
This function calls the get method from node's https package that we imported earlier as `getHttps`. It gets the result of this call as `res`, currently does northing with it, and will log any errors. So what do we do with this result? First of all, we need to deal with redirects. <https://opendata.rapid7.com/sonar.fdns_v2/2022-01-28-1643328400-fdns_a.json.gz>, The link on Project Sonar's site, actually redirects to backblaze, where the data is actually hosted, before allowing you to download it.

Fortunately, we can check if we need to redirect based on the result's [HTTP status code](https://httpstatuses.com/). If the status is 200, we're in the right place, and can return the result to be used elsewhere. If the status is 301 or 303, we should follow the redirect by calling the readFromWeb method again, with the new URL being passed in as an argument. I've added the following code inside the above `getHttps` call:
```
if (res.statusCode === 200) {
	parseSonar(client, res);
} else if (res.statusCode === 301 || res.statusCode === 302) {
	// Recursively follow redirects, only a 200 will resolve.
	console.log(`Redirecting to: ${res.headers.location}`);
	readFromWeb(client, res.headers.location);
} else {
	console.log(`Download request failed, response status: ${res.statusCode} ${res.statusMessage}`);
}
```
This function gets a read stream - not the actual data itself - that we can later read through and parse. We can then pass it to a function that does not yet exist (We'll add it shortly) along with the MongoClient this function was passed.

Now we can return to our main method and add in something to call our function
```
const dataUrl = "https://opendata.rapid7.com/sonar.fdns_v2/2022-01-28-1643328400-fdns_a.json.gz";
readFromWeb(client, dataUrl);
```

Note that if this doesn't work, make sure you have the latest link from <https://opendata.rapid7.com/sonar.fdns_v2/>, as downloading older/newer versions requires an account.

## Parsing our input
So now, using either of the above methods, we have a stream that will allow us to read in the project sonar data. Unfortunately, we still have two things to deal with before getting to anything useful: We have to get data out of the streams, and then we have to decompress the data we've been given - it's currently still gzipped.

Luckily, we can deal with both of those problems pretty quickly! Let's create a new function:
```
async function parseSonar(client, readstream) {
	// Pipe the response into gunzip to decompress
	let gunzip = zlib.createGunzip();

	let lineReader = readline.createInterface({
		input: readstream.pipe(gunzip),
	});
}
```

What we're doing here is:
 - Creating a writable stream called `gunzip` with `zlib`, node.js's module for compression/decompression
 - Piping our readstream of compressed Project Sonar data to this `gunzip` object
 - Taking the output of that, and using it as the input for a readline object, which allows us to parse the data one line at a time. (It also means we don't have to worry about buffers stopping mid-line.)

Quite a lot for a few lines of code! We also need to remember to call this new function from within our main function -   
Now, we still need to get our data out of this linereader. To do this, we can use the `"line"` event that the linereader emits to let us know when we have a new line to parse, with:
```
lineReader.on("line", (line) => {
	// We'll parse the line in here
});
```

So we've got a line of data - now what?  
The data is in JSON form, and luckily for us, we can simply use javascript's `JSON.parse()` to parse it. Next up, we need to break the hostname (eg `subdomain.example.com/path`) into it parts - we need just the `example` bit (This is required for performance - I'll explain more once we get to that point). We can do this pretty easily by using the `tldts` package's `parse` function we imported earlier as `tldParse`.

First, we need to deal with many records beginning with `*.`. If we don't remove this from the start of the hostname, we cannot properly parse it. Then, let's parse it with `tldParse` and log it, to make sure everything is working so far.

```
let lineJson = JSON.parse(line);
let hostname = lineJson.name;

if (hostname.substring(0, 2) === "*.") hostname = hostname.substring(2);

let tldParsed = tldParse(hostname);

console.log(tldParsed);
```

You should now hopefully see lines of JSON being printed! We should probably remove that `console.log` for now though - printing out every single line hurts our performance.  
Note that there are still a few invalid hostnames - Some beginning with `/`, `-` or `*`. I don't know why these are here, but given that only around 0.2% of the results are invalid, it's probably safe enough to ignore them for now.

## MongoDB
Now we need to start thinking about MongoDB. Whilst MongoDB is fast, it is unfortunately not fast enough to get us a quick result from 1.7 billion items. To speed it up, we'll make use of [text indexes](https://docs.mongodb.com/manual/core/index-text/).

Back in our main function, let's add a line to create this text index.  
```
await client.db("test_db").collection("sonardata").createIndex({ domainWithoutSuffix: "text" });
```
You can call your database and collection whatever you want - this is just what I'm using. We're using the domain without the suffix as our index, as that's what I'm wanting to query later on. If, however, you wanted to query IP address, to find out which domains point to a given IP address, you'd use it as your text index instead.

We also don't want redundant data building up each time we run our program - let's add something to drop the collection each time the program is run. (We don't need to add anything to create the collection again - MongoDB does this automatically for us whenever we try to add data to it.)
```
// Drop the collection containg Project Sonar data
try {
	await client.db("test_db").collection("sonardata").drop();
} catch {}
```

Nice! Now we can begin actually adding the data to MongoDB.  
Returning back to our parseSonar function - items can be inserted in bulk to MongoDB to increase performance, up to 100k items - so let's do that. After we've created the linereader, let's create an array and a counter to keep track of how many items we have.

Now, after the JSON has been parsed, we can increment our counter and add whatever data we want to our buffer array. Then, when our counter is evenly divisible by 100,000, we can log how many lines have been parsed, send our data to be added to MongoDB, and clear our buffer array. Our parseSonar function should now look something like:
```
async function parseSonar(client, readstream) {
	// Pipe the response into gunzip to decompress
	let gunzip = zlib.createGunzip();

	let lineReader = readline.createInterface({
		input: readstream.pipe(gunzip),
	});

	let arr = [];
	let count = 0;
	lineReader.on("line", (line) => {
		let lineJson = JSON.parse(line);
		let hostname = lineJson.name;
		if (hostname.substring(0, 2) === "*.") hostname = hostname.substring(2);

		let tldParsed = tldParse(hostname);

		if (tldParsed.domainWithoutSuffix) {
			count++;
			arr.push({
				domainWithoutSuffix: tldParsed.domainWithoutSuffix,
				publicSuffix: tldParsed.publicSuffix,
				subdomain: tldParsed.subdomain,
				name: lineJson.name,
				type: lineJson.type,
				value: lineJson.value,
			});
			
			if (count % 100000 === 0) {
				console.log(`${count} lines parsed`);
				createManyListings(client, arr, "sonardata");
				arr = [];
			}
		}
	});
}
```

Nearly done now! We just need to add the `createManyListings` function. Thankfully, it's pretty simple:
```
async function createManyListings(client, newListing, collection, dbName = "test_db") {
	client.db(dbName).collection(collection).insertMany(newListing, { ordered: false });
}
```
The only thing to note here is that we're telling MongoDB that our data is not/does not need to be ordered, helping increase our performance. Running the program now will begin filling up our database with data. Unfortunately, this is still a slow process - We have about 1.7 billion lines to parse!

## Querying MongoDB
So, we have our data sitting in a collection in MongoDB. Now what?

Create a new file called `queryData.js`. We can follow the basic template of the previous file to get started:
```
import { MongoClient } from "mongodb";

/**
 * Main function
 */
async function main() {
	// Database is currently hosted on same machine
	const uri = "mongodb://localhost:27017";
	const client = new MongoClient(uri);

	try {
		// Connect to the MongoDB cluster
		await client.connect();
		
		// Run query here
	} catch (e) {
		// Log any errors
		console.error(e);
	} finally {
		await client.close();
	}
}

// Run the main function
main().catch(console.error);
```

Now we need to come up with a query! As an example, I'll search for domains ...  
To actually query this, I'll use:
```
let query = { $text: { $search: "rapid7" }, domainWithoutSuffix: "rapid7" };
await findMany(client, query, "sonardata");
```
To explain what the query actually means:
 - `$text: { $search: "rapid7" }` is how we're able to make queries with a reasonable level of performance - It makes use of the text index we set up earlier, and matches with all  `domainWithoutSuffix`s that **contain** (*not* match exactly) the given query.
 - `domainWithoutSuffix: "rapid7"` narrows that down further to only the exact matches.

We could continue to further narrow this down if we wanted (For more info, see <https://docs.mongodb.com/manual/tutorial/query-documents/>). First though, we need to add in the `findMany` function that we're calling.

```
async function findMany(client, query, collection, db_name = "test_db", maxResults = 500) {
	const cursor = client.db(db_name).collection(collection).find(query).limit(maxResults);

	const results = await cursor.toArray();

	if (results.length > 0) {
		console.log("Found items:");
		results.forEach((result, i) => {
			console.log(result);
		});
	} else {
		console.log("No results found with the given query!");
	}
}
```

This function is fairly simple - all it does is fetch the results of the query to the given collection as an array, then if there are results, print them. And that's it! Now you're able to query Project Sonar's data! What you do with the results you get are up to you.