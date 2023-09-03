<script lang="ts">
	import Button, { Label } from '@smui/button';
	import initSqlJs from 'sql.js';
	// Create a global variable to store the database

	// let _db: initSqlJs.Database | undefined;
	let _db: initSqlJs.Database;

	// A function that opens a database
	async function openDB() {
		console.log('Opening DB');

		const SQL = await initSqlJs({
			// Required to load the wasm binary asynchronously. Of course, you can host it wherever you want
			// You can omit locateFile completely when running in node
			locateFile: (file: any) => `https://sql.js.org/dist/${file}`
		});

		// Create a database
		_db = new SQL.Database();
	}

	async function closeDB() {
		console.log('Closing DB');

		if (_db === undefined) {
			console.error('DB is undefined');
			return;
		}

		// Close the database connection
		_db.close();
	}

	// A temp function that does nothing
	async () => {
		try {
			_db.run(`
            INSERT INTO images (path, name, description)
            VALUES
                ('/images/1.jpg', 'Image 1', 'This is the first image'),
                ('/images/2.jpg', 'Image 2', 'This is the second image'),
                ('/images/3.jpg', 'Image 3', 'This is the third image');
        `);
		} catch (e) {
			console.error('exception inserting data: ', e);
			return;
		}

		try {
			// Prepare a statement
			const stmt = _db.prepare(`
            SELECT * 
            FROM images;
        `);

			stmt.getAsObject({ $start: 1, $end: 1 }); // {col1:1, col2:111}

			// Bind new values
			stmt.bind({ $start: 1, $end: 2 });
			while (stmt.step()) {
				//
				const row = stmt.getAsObject();
				console.log('Here is a row: ' + JSON.stringify(row));
			}
		} catch (e) {
			console.error('exception selecting data: ', e);
			return;
		}
	};

	async function initDB() {
		console.log('Initializing DB');

		if (_db === undefined) {
			console.error('DB is undefined');
			return;
		}

		try {
			// Create a table "images" with the columns, id, path, title, description
			_db.run(`
				CREATE TABLE images (
					id INTEGER PRIMARY KEY AUTOINCREMENT,
					path TEXT,
					name TEXT,
					description TEXT
				);
			`);
			// Create a table "tagmap" with the columns, id, image_id, tag_id
			_db.run(`
				CREATE TABLE tagmap (
					id INTEGER PRIMARY KEY AUTOINCREMENT,
					image_id INTEGER,
					tag_id INTEGER
				);
			`);
			// Create a table "tags" with the columns, id, name
			_db.run(`
				CREATE TABLE tags (
					id INTEGER PRIMARY KEY AUTOINCREMENT,
					name TEXT
				);
			`);
		} catch (e) {
			console.error('exception creating tables: ', e);
			return;
		}
	}

	// A function that takes a list of image paths and inserts them into the database
	async function insertImages(imagePaths: string[]) {
		console.log('Inserting images');

		if (_db === undefined) {
			console.error('DB is undefined');
			return;
		}

		try {
			// Prepare a statement
			const stmt = _db.prepare(`
				INSERT INTO images (path, name, description)
				VALUES
					(:path, :name, :description);
			`);

			// Bind values to the parameters and execute the statement
			for (const imagePath of imagePaths) {
				stmt.run({
					':path': imagePath,
					':name': 'Image Name',
					':description': 'Image Description'
				});
			}
		} catch (e) {
			console.error('exception inserting data: ', e);
			return;
		}
	}

	async function testInsertImages() {
		const someImagePaths: Array<string> = ['/images/1.jpg', '/images/2.jpg', '/images/3.jpg'];
		await insertImages(someImagePaths);
	}

	async function listImages() {
		console.log('Listing images');

		if (_db === undefined) {
			console.error('DB is undefined');
			return;
		}

		try {
			// Prepare a statement
			const stmt = _db.prepare(`
				SELECT * 
				FROM images;
			`);

			stmt.getAsObject({ $start: 1, $end: 1 }); // {col1:1, col2:111}

			// Bind new values
			stmt.bind({ $start: 1, $end: 2 });
			while (stmt.step()) {
				//
				const row = stmt.getAsObject();
				console.log('Here is a row: ' + JSON.stringify(row));
			}
		} catch (e) {
			console.error('exception selecting data: ', e);
			return;
		}
	}
</script>

<h1>Testing sql.js</h1>

<Button on:click={openDB}>
	<Label>Open DB</Label>
</Button>

<Button on:click={initDB}>
	<Label>Init DB</Label>
</Button>

<Button on:click={testInsertImages}>
	<Label>Insert Images</Label>
</Button>

<Button on:click={listImages}>
	<Label>List Images</Label>
</Button>
