<script context="module" lang="ts">
	import initSqlJs from 'sql.js';

	export type image = {
		title: string;
		description: string;
		path: string;
	};

	let _db: initSqlJs.Database;

	// A function that opens a database
	export async function openDB() {
		console.log('Opening DB');

		const SQL = await initSqlJs({
			// Required to load the wasm binary asynchronously. Of course, you can host it wherever you want
			// You can omit locateFile completely when running in node
			locateFile: (file: any) => `https://sql.js.org/dist/${file}`
		});

		// Create a database
		_db = new SQL.Database();
	}

	export async function closeDB() {
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
            INSERT INTO images (path, title, description)
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

	export async function initDB() {
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
					title TEXT,
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
	export async function insertImages(imagePaths: string[]) {
		console.log('Inserting images');

		if (_db === undefined) {
			console.error('DB is undefined');
			return;
		}

		try {
			// Prepare a statement
			const stmt = _db.prepare(`
				INSERT INTO images (path, title, description)
				VALUES
					(:path, :title, :description);
			`);

			// Bind values to the parameters and execute the statement
			let i = 1;
			for (const imagePath of imagePaths) {
				stmt.run({
					':path': imagePath,
					':title': 'Image Title' + i,
					':description': 'Image Description ' + i++
				});
			}
		} catch (e) {
			console.error('exception inserting data: ', e);
			return;
		}
	}

	export async function testInsertImages() {
		const someImagePaths: Array<string> = ['/images/1.jpg', '/images/2.jpg', '/images/3.jpg'];
		await insertImages(someImagePaths);
	}

	export async function listImages(): Promise<Array<image>> {
		console.log('Listing images');

		if (_db === undefined) {
			console.error('DB is undefined');
			return [];
		}

		try {
			// Prepare a statement
			const stmt = _db.prepare(`
				SELECT * 
				FROM images;
			`);

			stmt.getAsObject({ $start: 1, $end: 1 }); // {col1:1, col2:111}

			// declare images to be an array of image objects
			let images: Array<image> = [];

			// Bind new values
			stmt.bind({ $start: 1, $end: 2 });
			while (stmt.step()) {
				//
				const row = stmt.getAsObject();
				console.log('Here is a row: ' + JSON.stringify(row));
				const nextTitle: string = row.title?.toLocaleString()!;
				const nextDescription: string = row.description?.toLocaleString()!;
				const nextPath: string = row.path?.toLocaleString()!;
				images.push({ title: nextTitle, description: nextDescription, path: nextPath });
			}

			return images;
		} catch (e) {
			console.error('exception selecting data: ', e);
			return [];
		}
	}
</script>
