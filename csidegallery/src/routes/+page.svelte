<script lang="ts">
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

	async function initDB() {
		console.log('Initializing DB');

		if (_db === undefined) {
			console.error('DB is undefined');
			return;
		}

		try {
			// Create a table "images" with the columns, path, name, descrption, and tags
			_db.run(`
            CREATE TABLE images (
                path TEXT,
                name TEXT,
                description TEXT
            );
        `);
		} catch (e) {
			console.error('exception creating table: ', e);
			return;
		}

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
	}
</script>

<h1>Testing sql.js</h1>

<button on:click={openDB}>Open DB</button>

<button on:click={initDB}>Init DB</button>
