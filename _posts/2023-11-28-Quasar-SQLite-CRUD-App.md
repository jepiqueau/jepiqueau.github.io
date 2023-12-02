
# Quasar SQLite Database CRUD App Example Tutorial using @capacitor-community/sqlite
---

*last updated on December 2, 2023 by Quéau Jean Pierre*

In that tutorial we will learned how to create a Quasar basic CRUD application and implement the @capacitor-community/sqlite plugin to store the data in a SQLite database.

The first part of the tutorial will concentrate on: 
 - how to create that application and run it on a Web browser where the data will be stored on an Indexed database using sql.js and localForage modules.
 - how to use the upgrade statement to create a version 2 of the database by altering the datacase's schema.

Go to [Part 1 - Web - Table of Contents](#part-1---web---table-of-contents)

The application can be found at [Part-1/quasar-sqlite-app](https://github.com/jepiqueau/blog-tutorials-apps/tree/main/SQLite/Part-1/quasar-sqlite-app)


Thanks to the Ionic Team and their hard work to bring CAPACITOR 5, the second part will concentrate on native platforms (iOS and Android) and also on Electron platform.
Go to [Part 2 - Native - Table of Contents](#part-2---native---table-of-contents)

The application can be found at [Part-2/quasar-sqlite-app](https://github.com/jepiqueau/blog-tutorials-apps/tree/main/SQLite/Part-2/quasar-sqlite-app)



## Part 1 - Web - Table of Contents

---
 - [Install New Quasar Application](#install-new-quasar-application)
 - [Install Capacitor](#install-capacitor)
 - [Install Required Packages](#install-required-packages)
 - [Create the SQLite Database](#create-the-sqlite-database)
 - [Create CRUD and SQLite Services](#create-crud-and-sqlite-services)
 - [Quasar Plugin Implementation](#quasar-plugin-implementation)
 - [Modify the Home Page](#modify-the-home-page)
 - [Modify the Main Layout](#modify-the-main-layout)
 - [Users Page Implementation](#users-page-implementation)
 - [Users Component Implementation](#users-component-implementation)
 - [Create the QuerySQLite Hook](#create-the-querysqlite-hook)
 - [Update Config](#update-config)
 - [Run the Web SQLite App](#run-the-web-sqlite-app)
 - [Video Storyboard](#video-storyboard)
 - [Upgrade Database to Version 2](#upgrade-database-to-version-2)
 - [Part 1 Conclusion](#part-1-conclusion)

---

### Install New Quasar Application

 - Install the latest version of Quasar CLI globally installed on your device, run the below command.

    ```bash
    npm install -g @quasar/cli
    ```

 - Create a new Quasar app

    ```bash
    npm init quasar
    ```

    You’ll be prompted with some options:

    - App with Quasar CLI, let's go!
    - Project folder: `quasar-sqlite-app`
    - Pick Quasar Version: `Quasar V2 (Vue 3 | latest and greatest)`
    - Pick Script Type: `Typescript`
    - Pick Quasar App Cli variant: `Quasar App CLI with Vite`
    - Package name: `quasar-sqlite-app` 
    - Project product name: `Quasar SQLite App`
    - Project description: `A Quasar SQLite Capacitor Project`
    - Author: `YOUR_NAME`
    - Pick a Vue component style: `Composition API`
    - Pick your CSS preprocessor: `None`
    - Check the features needed for your project: `ESLint` 
    - Pick an ESLint preset: `Prettier`
    - Install project dependencies?: `Yes, use npm`

  - Test the application

  ```bash
  cd quasar-sqlite-app
  npm run dev
  ```
  Opening default browser at http://localhost:9000/

  Yow must see a Quasar App with a menu `Essentials Links` and an `Example component`.

### Install Capacitor

 You must install the latest version of `@capacitor/core` and `@capacitor/cli`.
 
 ```bash
    npm i @capacitor/core
    npm i -D @capacitor/cli
 ```

 Then, initialize your Capacitor config.

 ```bash
    npx cap init
 ```

 You’ll be prompted with some options:

 - What is the name of your app?: `quasar-sqlite-app`
 - What should be the Package ID for your app?: `YOUR_PACKAGE_ID` 
 - What is the web asset directory for your app?: `dist/spa`

 When done, you must get [success] capacitor.config.ts created!

### Install Required Packages

 - run the below commands

    ```bash
    npm i @capacitor-community/sqlite
    npm i @capacitor/toast
    npm i @ionic/pwa-elements
    npm i -D copyfiles
    ```

### Create the SQLite Database

 - To easily upgrade the database from one version to the next, the database is created through an upgrade statement. In your editor, create a folder `upgrades` under the `src` folder and then create a new file `upgrades/user.statements.ts`.

 - Define the upgrade statements in `upgrades/user.upgrade.statements.ts` file

    ```ts
    export const UserUpgradeStatements = [
        {
            toVersion: 1,
            statements: [
                `CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                active INTEGER DEFAULT 1
                );`
            ]
        },
        /* add new statements below for next database version when required*/
        /*
        {
            toVersion: 2,
            statements: [
                `ALTER TABLE users ADD COLUMN email TEXT;`,
            ]
        },
        */
    ]
   
    ```

 - Define the User Model

    In your editor, create a folder `models` under the `src` folder and then create a new file `models/user.ts`.

 - Update the `models/user.ts` file as

    ```ts
    export interface User {
        id: number;
        name: string;
        active: number;
        /*
        email: string; // Version 2
        */
    }
    ```

### Create CRUD and SQLite Services

 The CRUD service will be dedicated to the CRUD operations of the application like:

  - addUser,
  - deleteUserById,
  - getDatabaseName,
  - getDatabaseVersion,
  - getUsers,
  - initializeDatabase,
  - replaceUser,
  - updateUserActiveById,

  While the SQLite service will be dealing with the API to the `@capacitor-community/sqlite` plugin including:

  - addUpgradeStatement,
  - closeDatabase,
  - getPlatform,
  - initWebStore,
  - isConnection,
  - openDatabase,
  - saveToStore,
  - saveToLocalDisk,

 - To create CRUD and SQLite services, in your file editor, add a new `src/services` folder and create the two following files:

    - `services/sqliteService.ts`
    - `services/storageService.ts`

 - Open the `services/sqliteService.ts` file and replace with the following code.

    ```ts
    import {
    CapacitorSQLite,
    SQLiteConnection,
    SQLiteDBConnection,
    capSQLiteUpgradeOptions,
    } from '@capacitor-community/sqlite';
    import { Capacitor } from '@capacitor/core';

    export interface ISQLiteService {
        addUpgradeStatement(options: capSQLiteUpgradeOptions): Promise<void>;
        closeDatabase(dbName: string, readOnly: boolean): Promise<void>;
        getPlatform(): string;
        initWebStore(): Promise<void>;
        isConnection(dbName: string, readOnly: boolean): Promise<boolean>;
        openDatabase(
            dbName: string,
            loadToVersion: number,
            readOnly: boolean
        ): Promise<SQLiteDBConnection | undefined>;
        saveToStore(dbName: string): Promise<void>;
        saveToLocalDisk(dbName: string): Promise<void>;
    }

    class SQLiteService implements ISQLiteService {
        platform = Capacitor.getPlatform();
        sqlitePlugin = CapacitorSQLite;
        sqliteConnection = new SQLiteConnection(CapacitorSQLite);
        dbNameVersionDict: Map<string, number> = new Map();

        async addUpgradeStatement(options: capSQLiteUpgradeOptions): Promise<void> {
            // add the upgrade statement to define the database schema
            // depending on the database version
            try {
            await this.sqlitePlugin.addUpgradeStatement(options);
            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            } catch (error: any) {
            const msg = error.message ? error.message : error;
            throw new Error(`sqliteService.addUpgradeStatement: ${msg}`);
            }
            return;
        }

        async closeDatabase(dbName: string, readOnly: boolean): Promise<void> {
            // close the connection to the database and close the database
            try {
            const isConn = (
                await this.sqliteConnection.isConnection(dbName, readOnly)
            ).result;
            if (isConn) {
                await this.sqliteConnection.closeConnection(dbName, readOnly);
            }
            return;
            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            } catch (error: any) {
            const msg = error.message ? error.message : error;
            throw new Error(`sqliteService.closeDatabase: ${msg}`);
            }
        }

        getPlatform(): string {
            // return the device platform
            return this.platform;
        }

        async initWebStore(): Promise<void> {
            // initialize the web store to store the database for web platform
            try {
            await this.sqliteConnection.initWebStore();
            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            } catch (error: any) {
            const msg = error.message ? error.message : error;
            throw new Error(`sqliteService.initWebStore: ${msg}`);
            }
            return;
        }
        async isConnection(dbName: string, readOnly: boolean): Promise<boolean> {
            // check if a connection exist to the database
            try {
            const isConn = (
                await this.sqliteConnection.isConnection(dbName, readOnly)
            ).result;
            if (isConn != undefined) {
                return isConn;
            } else {
                throw new Error('sqliteService.isConnection undefined');
            }
            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            } catch (error: any) {
            const msg: string = error.message ? error.message : error;
            throw new Error(`sqliteService.isConnection: ${msg}`);
            }
        }

        async openDatabase(
            dbName: string,
            loadToVersion: number,
            readOnly: boolean
        ): Promise<SQLiteDBConnection | undefined> {
            // create a database connection and open the database
            this.dbNameVersionDict.set(dbName, loadToVersion);
            const encrypted = false;
            const mode = encrypted ? 'secret' : 'no-encryption';
            try {
            let db: SQLiteDBConnection;
            const retCC = (await this.sqliteConnection.checkConnectionsConsistency())
                .result;
            const isConn = (
                await this.sqliteConnection.isConnection(dbName, readOnly)
            ).result;
            if (retCC && isConn) {
                db = await this.sqliteConnection.retrieveConnection(dbName, readOnly);
            } else {
                db = await this.sqliteConnection.createConnection(
                dbName,
                encrypted,
                mode,
                loadToVersion,
                readOnly
                );
            }
            await db.open();
            const res = await db.isDBOpen();
            if (!res.result) {
                const msg = 'database is not opened';
                throw new Error(`sqliteService.openDatabase: ${msg}`);
            }
            return db;
            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            } catch (error: any) {
            const msg = error.message ? error.message : error;
            throw new Error(`sqliteService.openDatabase: ${msg}`);
            }
        }

        async saveToLocalDisk(dbName: string): Promise<void> {
            // save the database to local disk for the Web platform
            try {
            await this.sqliteConnection.saveToLocalDisk(dbName);
            return;
            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            } catch (err: any) {
            const msg = err.message ? err.message : err;
            throw new Error(`sqliteService.saveToLocalDisk: ${msg}`);
            }
        }

        async saveToStore(dbName: string): Promise<void> {
            // save the database to store for the Web platform
            try {
            await this.sqliteConnection.saveToStore(dbName);
            return;
            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            } catch (error: any) {
            const msg = error.message ? error.message : error;
            throw new Error(`sqliteService.saveToStore: ${msg}`);
            }
        }
    }
    export default SQLiteService;
    ```

 - Open the `services/storage.service.ts` and replace the code with

    ```ts
    import { BehaviorSubject } from 'rxjs';
    import { getCurrentInstance } from 'vue';
    import { ISQLiteService } from '../services/sqliteService';
    import { IDbVersionService } from '../services/dbVersionService';
    import { SQLiteDBConnection } from '@capacitor-community/sqlite';
    import { UserUpgradeStatements } from '../upgrades/user.statements';
    import { User } from '../models/User';

    export interface IStorageService {
        addUser(user: User): Promise<number>;
        deleteUserById(id: string): Promise<void>;
        getDatabaseName(): string;
        getDatabaseVersion(): number;
        getUsers(): Promise<User[]>;
        initializeDatabase(): Promise<void>;
        replaceUser(user: User): Promise<void>;
        updateUserActiveById(id: string, active: number): Promise<void>;
    }
    class StorageService implements IStorageService {
        versionUpgrades = UserUpgradeStatements;
        loadToVersion =
            UserUpgradeStatements[UserUpgradeStatements.length - 1].toVersion;
        db!: SQLiteDBConnection | undefined;
        database = 'myuserdb';
        sqliteServ!: ISQLiteService;
        dbVerServ!: IDbVersionService;
        isInitCompleted = new BehaviorSubject(false);
        appInstance = getCurrentInstance();
        platform!: string;

        constructor(
            sqliteService: ISQLiteService,
            dbVersionService: IDbVersionService
        ) {
            this.sqliteServ = sqliteService;
            this.dbVerServ = dbVersionService;
            this.platform =
            this.appInstance?.appContext.config.globalProperties.$platform;
        }

        async addUser(user: User): Promise<number> {
            // add a user to the database
            const colList = Object.keys(user).slice(1).toString();
            const valArr = Object.values(user).slice(1);
            const valList: string = valArr
                .map((value) => (typeof value === 'string' ? `'${value}'` : value))
                .join(',');

            const sql = `INSERT INTO users (${colList}) VALUES (${valList});`;
            const res = await this.db?.run(sql, []);
            if (
                res?.changes !== undefined &&
                res.changes.lastId !== undefined &&
                res.changes.lastId > 0
            ) {
                return res.changes.lastId;
            } else {
                throw new Error('storageService.addUser: lastId not returned');
            }
        }

        async deleteUserById(id: string): Promise<void> {
            // delete a user by id from the database
            const sql = `DELETE FROM users WHERE id=${id}`;
            await this.db?.run(sql);
        }

        getDatabaseName(): string {
            // return the database name
            return this.database;
        }

        getDatabaseVersion(): number {
            // return the database version
            return this.loadToVersion;
        }

        async getUsers(): Promise<User[]> {
            // return all users
            return (await this.db?.query('SELECT * FROM users;'))?.values as User[];
        }

        async initializeDatabase(): Promise<void> {
            // initialize the database
            try {
                // create upgrade statements
                await this.sqliteServ.addUpgradeStatement({
                    database: this.database,
                    upgrade: this.versionUpgrades,
                });
                // open the connection to the database with the latest version
                // and then open the database
                this.db = await this.sqliteServ.openDatabase(
                    this.database,
                    this.loadToVersion,
                    false
                );
                const isData = await this.db?.query('select * from sqlite_sequence');
                if (isData?.values && isData.values.length === 0) {
                    // create database initial users if any
                }
                // store the database version
                this.dbVerServ.setDbVersion(this.database, this.loadToVersion);
                if (this.platform === 'web') {
                    await this.sqliteServ.saveToStore(this.database);
                }
                this.isInitCompleted.next(true);
            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            } catch (error: any) {
                const msg = error.message ? error.message : error;
                throw new Error(`storageService.initializeDatabase: ${msg}`);
            }
        }

        async replaceUser(user: User): Promise<void> {
            const colList = Object.keys(user).toString();
            const valArr = Object.values(user);
            const valList: string = valArr
                .map((value) => (typeof value === 'string' ? `'${value}'` : value))
                .join(',');
            const sql = `INSERT OR REPLACE INTO users (${colList}) VALUES (${valList});`;
            const res = await this.db?.run(sql, []);
            if (
                res?.changes !== undefined &&
                res.changes.lastId !== undefined &&
                res.changes.lastId > 0
            ) {
                return;
            } else {
                throw new Error('storageService.replaceUser: lastId not returned');
            }
        }

        async updateUserActiveById(id: string, active: number): Promise<void> {
            const sql = `UPDATE users SET active=${active} WHERE id=${id}`;
            await this.db?.run(sql);
        }
    }
    export default StorageService;
    ```    

 To manage the database versionning, a new service `DbnameVersionService` is required. In your file editor, go to `src/services` folder and create the `dbVersionService.ts` file


 - Open the `services/dbVersionService.ts` file and replace the code with

    ```ts
    export interface IDbVersionService {
        getDbVersion(dbName: string):number| undefined
        setDbVersion(dbName: string, version: number): void
    };
    class DbVersionService implements IDbVersionService  {
        dbNameVersionDict: Map<string, number> = new Map();

        getDbVersion(dbName: string): number | undefined {
            const version =  this.dbNameVersionDict.get(dbName);
            return version;
        };

        setDbVersion(dbName: string, version: number) {
            this.dbNameVersionDict.set(dbName, version);
        };
    }
    export default new DbVersionService();
    ```

### Quasar Plugin Implementation

 For Web applications, the plugin uses the Stencil component `jeep-sqlite` which implies to define it in  App.vue file and to copy the `sql-wasm.wasm`file to the `public/assets`folder.

 - Open the `package.json` and modify the script as follows:

 ```json
   "scripts": {
        "lint": "eslint --ext .js,.ts,.vue ./",
        "format": "prettier --write \"**/*.{js,ts,vue,css,html,md,json}\" --ignore-path .gitignore",
        "test": "echo \"No test specified\" && exit 0",
        "dev": "npm run copy:sql:wasm && quasar dev",
        "build:web": "npm run copy:sql:wasm && quasar build",
        "copy:sql:wasm": "copyfiles -u 3 node_modules/sql.js/dist/sql-wasm.wasm public/assets"
    },
``` 

 As the goal is to ensure that the services (`SqliteService`, `DbVersionService` and `StorageService`) are unique and properly shared and available throughout the components, they have been defined in the App.vue file, to be injected in components.

 During the mounting of the app an `InitializeAppService` service is used to initialize the application and has to be created.

 - In your Editor add a new file `initializeAppService.ts` file under the folder `src/services`.

 - In your editor, open the file `services/initializeAppService.ts` and replace the code with:

    ```ts
    import { ISQLiteService } from '../services/sqliteService';
    import { IStorageService } from '../services/storageService';

    export interface IInitializeAppService {
    initializeApp(): Promise<boolean>;
    }

    class InitializeAppService implements IInitializeAppService {
        appInit = false;
        sqliteServ!: ISQLiteService;
        storageServ!: IStorageService;
        platform!: string;

        constructor(sqliteService: ISQLiteService, storageService: IStorageService) {
            this.sqliteServ = sqliteService;
            this.storageServ = storageService;
            this.platform = this.sqliteServ.getPlatform();
        }
        async initializeApp(): Promise<boolean> {
            if (!this.appInit) {
                try {
                    if (this.platform === 'web') {
                    await this.sqliteServ.initWebStore();
                    }
                    // Initialize the myuserdb database
                    await this.storageServ.initializeDatabase();
                    if (this.platform === 'web') {
                    await this.sqliteServ.saveToStore(this.storageServ.getDatabaseName());
                    }
                    this.appInit = true;
                    // eslint-disable-next-line @typescript-eslint/no-explicit-any
                } catch (error: any) {
                    const msg = error.message ? error.message : error;
                    throw new Error(`initializeAppError.initializeApp: ${msg}`);
                }
            }
            return this.appInit;
        }
    }
    export default InitializeAppService;
   ```

 - Open the `App.vue` file  and replace the code with

    ```ts
    <template>
        <router-view />
    </template>

    <script lang="ts">
        import { defineComponent, onMounted } from 'vue';
        import { Capacitor } from '@capacitor/core';
        import { defineCustomElements as pwaElements } from '@ionic/pwa-elements/loader';
        import { JeepSqlite } from 'jeep-sqlite/dist/components/jeep-sqlite';
        import SqliteService from './services/sqliteService';
        import DbVersionService from './services/dbVersionService';
        import StorageService from './services/storageService';
        import InitializeAppService from './services/initializeAppService';

        // initialize pwaElements for Toast in Web platform
        pwaElements(window);
        // Define services as unique
        const sqliteServ = new SqliteService();
        const dbVersionServ = new DbVersionService();
        const storageServ = new StorageService(sqliteServ, dbVersionServ);

        export default defineComponent({
            name: 'App',
            provide: {
                sqliteServ,
                dbVersionServ,
                storageServ,
            },

            setup() {
                customElements.define('jeep-sqlite', JeepSqlite);
                const platform = Capacitor.getPlatform();
                //Define and instantiate the InitializeAppService
                const initAppServ = new InitializeAppService(sqliteServ, storageServ);

                const initApp = async () => {
                    try {
                        await initAppServ.initializeApp();
                    } catch (error) {
                        console.error('App Initialization error:', error);
                    }
                };
                onMounted(async () => {
                    if (platform != 'web') {
                        await initApp();
                    } else {
                        const jeepEl = document.createElement('jeep-sqlite');
                        document.body.appendChild(jeepEl);
                        customElements
                        .whenDefined('jeep-sqlite')
                        .then(async () => {
                            await initApp();
                        })
                        .catch((err) => {
                            console.error('jeep-sqlite creation error:', err);
                        });
                    }
                });
                return {};
            },
        });
    </script>
    ``` 

### Modify the Home Page

 The `Index` page will be modify to include:

    - a company logo
    - a welcome text

 This will be achieved by creating new components that will be integrated in the `Index` page.

 - Edit the `IndexPage.vue` file under the `pages` folder.

 ```ts
    <template>
    <q-page>
        <app-logo></app-logo>
        <app-intro-text></app-intro-text>
    </q-page>
    </template>

    <script lang="ts">
        import { defineComponent } from 'vue';
        import AppLogo from 'components/AppLogo.vue';
        import AppIntroText from 'components/AppIntroText.vue';

        export default defineComponent({
        name: 'IndexPage',
        components: { AppLogo, AppIntroText },
        setup() {
            return {};
        },
        });
    </script>
 ```

 - Create an `AppLogo` component

    - In your Editor create a `AppLogo.vue` file under `src/components`folder and copy the following code

        ```ts
        <template>
            <div class="AppLogo">
            <img src="/assets/JeepQLogo.png" alt="App Logo" width="128" height="128" />
            </div>
        </template>
        
        <script lang="ts">
            import { defineComponent } from 'vue';
            export default defineComponent({
                name: 'AppLogo',
            });
        </script>

        <style scoped>
            .AppLogo {
                width: 100%;
                display: flex;
                justify-content: center;
                margin-top: 2rem;
                margin-bottom: 1rem;
            }
        </style>
        ```

 Obviously use your own logo and store it in the `public/assets` folder.

 - Create an `AppIntroText` component

    - In your Editor create a `AppIntroText.vue` file under `src/components`folder and copy the following code

        ```ts
        <template>
            <div class="AppIntroText">
                <h3>Welcome to the Quasar SQLite Database CRUD App Example Tutorial</h3>
                <h5>using @capacitor-community/sqlite</h5>
            </div>
        </template>

        <script lang="ts">
            import { defineComponent } from 'vue';
            export default defineComponent({
                name: 'AppIntroText',
            });
        </script>

        <style scoped>
            .AppIntroText {
                width: 100%;
                display: flex;
                flex-direction: column;
                justify-content: center;
                text-align: center;
                padding: 0.5rem;
            }
        </style>
    ```

 Fill free to put whatever text you would like to see

 - Delete the `ExampleComponent.vue`component and the `models.ts`from the `src/components`folder

### Modify the Main Layout

The main layout will include a menu on the right, which will allow users to navigate through the application's pages and return to the Home page.

 - Edit the `MainLayout.vue` file under the `src/layouts` folder and replace the code as below.

 ```ts
    <template>
    <q-layout ref="layout" view="lHh Lpr lFf">
        <q-header elevated>
        <q-toolbar>
            <q-btn v-if="isBack" @click="goBack">Back</q-btn>
            <q-toolbar-title>{{ pageTitle }} </q-toolbar-title>
            <q-btn
            flat
            dense
            round
            icon="menu"
            aria-label="Menu"
            @click="toggleRightDrawer"
            />
        </q-toolbar>
        </q-header>

        <q-drawer v-model="rightDrawerOpen" side="right" show-if-above bordered>
        <q-list>
            <q-item-label header> Menu Content </q-item-label>

            <MenuRoute v-for="page in pages" :key="page.title" v-bind="page" />
        </q-list>
        </q-drawer>

        <q-page-container>
        <router-view />
        </q-page-container>
    </q-layout>
    </template>

    <script lang="ts">
    import { defineComponent, ref } from 'vue';
    import MenuRoute from 'components/MenuRoute.vue';

    const pageList = [
        {
            title: 'Managing Users',
            name: 'users',
        },
    ];

    export default defineComponent({
        name: 'MainLayout',
        components: {
            MenuRoute,
        },

        setup() {
            const rightDrawerOpen = ref(false);
            return {
            pages: pageList,
            rightDrawerOpen,
            toggleRightDrawer() {
                rightDrawerOpen.value = !rightDrawerOpen.value;
            },
            };
        },
        data() {
            return {
            pageTitle: 'Quasar SQLite App', // Set your default title here
            isBack: false,
            };
        },
        watch: {
            // Watch for changes in the route and update the pageTitle accordingly
            $route(to) {
            if (to.name !== 'home') this.isBack = true;
            this.pageTitle = to.meta.title || 'Quasar SQLite App';
            },
        },
        methods: {
            goBack() {
            this.isBack = false;
            this.$router.push({ name: 'home' });
            },
        },
    });
    </script>
 ```

 - Create the `MenuRoute.vue` component under the `src/components` and add the following code.

 ```ts
    <template>
    <q-item clickable @click="onClick()">
        <q-item-section>
        <q-item-label>{{ title }}</q-item-label>
        </q-item-section>
    </q-item>
    </template>

    <script lang="ts">
    import { defineComponent } from 'vue';

    export default defineComponent({
        name: 'MenuRoute',
        props: {
            title: {
            type: String,
            required: true,
            },

            name: {
            type: String,
            required: true,
            },
        },
        methods: {
            onClick() {
            this.$router.push({ name: this.$props.name });
            },
        },
    });
    </script>
 ```

 - Delete the `EssentialLink.vue`component in the `src/components`folder.

### Users Page Implementation

 The Users Page design will allow access to the CRUD operations. 
 Let's create it.

 - In your Editor, under `src/pages` folder add a new page `UsersPage.vue`.

 - Then open the `UsersPage.vue` file and add the following code:

    ```ts
    <template>
        <q-page>
            <div v-if="isInitComplete && dbInitialized">
            <q-card>
                <q-card-section>
                <div class="text-h5">Add New User</div>
                <user-form method="add" :onAddUser="handleAddUser"></user-form>
                </q-card-section>
            </q-card>
            <q-card>
                <q-card-section>
                <div class="text-h5">Current Users</div>
                <user-list
                    :users="users"
                    :onUpdateUserActive="handleUpdateUserActive"
                    :onDeleteUser="handleDeleteUser"
                    :onEditUser="handleEditUser"
                />
                </q-card-section>
            </q-card>
            <!-- q-dialog for the centered inline modal content -->
            <q-dialog v-model="centeredModalOpen">
                <q-card style="width: 60vw">
                <q-card-section>
                    <!-- Modal content goes here -->
                    <div class="text-h5">Update User</div>
                    <user-form
                    method="update"
                    :onUpdateUser="handleUpdateUser"
                    :user="editUser"
                    ></user-form>
                </q-card-section>
                </q-card>
            </q-dialog>
            </div>
        </q-page>
    </template>

    <script lang="ts">
    import {
    defineComponent,
    ref,
    inject,
    computed,
    onMounted,
    onBeforeUnmount,
    watch,
    } from 'vue';
    import UserForm from 'components/UserForm.vue';
    import UserList from 'components/UserList.vue';
    import { User } from '../models/User';
    import { SQLiteDBConnection } from '@capacitor-community/sqlite';
    import { Toast } from '@capacitor/toast';
    import { useQuerySQLite } from '../hooks/UseQuerySQLite';
    import SQLiteService from 'src/services/sqliteService';
    import StorageService from 'src/services/storageService';

    export default defineComponent({
        name: 'UsersPage',
        components: { UserForm, UserList },
        setup() {
            // Inject services
            const sqliteServ: SQLiteService | undefined = inject('sqliteServ');
            const storageServ: StorageService | undefined = inject('storageServ');
            // Define Internal variables
            const centeredModalOpen = ref(false);
            const centeredModalMarginTop = ref('20vh');
            const db = ref<SQLiteDBConnection | undefined>(undefined);
            const dbInitialized = computed(() => !!db.value);
            const dbNameRef = ref('');
            const editUser = ref<User>({} as User);
            const isInitComplete = ref(false);
            const isDatabase = ref(false);
            const platform = sqliteServ?.getPlatform();
            const users = ref<User[]>([]);

            // Define methods
            const getAllUsers = async (db: SQLiteDBConnection) => {
                // Query the database to get all users
                const stmt = 'SELECT * FROM users';
                const values: never[] = [];
                const fetchData = await useQuerySQLite(db, stmt, values);
                users.value = fetchData;
            };

            const openDatabase = async () => {
                // Open the connection to the database
                try {
                    const dbUsersName = storageServ?.getDatabaseName() as string;
                    dbNameRef.value = dbUsersName as string;
                    const version = storageServ?.getDatabaseVersion() as number;

                    const database = await sqliteServ?.openDatabase(
                    dbUsersName,
                    version,
                    false
                    );
                    db.value = database;
                    isDatabase.value = true;
                } catch (error) {
                    const msg = `Error open database: ${error}`;
                    console.error(msg);
                    Toast.show({
                        text: msg,
                        duration: 'long',
                    });
                }
            };

            const handleAddUser = async (newUser: User) => {
                // Store the new user in the database
                if (db.value) {
                    const isConn = await sqliteServ?.isConnection(dbNameRef.value, false);
                    if (!isConn) {
                        const msg = 'Error handleAddUser: No DatabaseConnection';
                        console.error(msg);
                        Toast.show({
                            text: msg,
                            duration: 'long',
                        });
                    }
                    const lastId = await storageServ?.addUser(newUser);
                    newUser.id = lastId as number;
                    users.value.push(newUser as never);
                }
            };

            const handleUpdateUser = async (updateUser: User) => {
                // Store the updated user data in the database
                if (db.value) {
                    const isConn = await sqliteServ?.isConnection(dbNameRef.value, false);
                    if (!isConn) {
                        const msg = 'Error handleAddUser: No DatabaseConnection';
                        console.error(msg);
                        Toast.show({
                            text: msg,
                            duration: 'long',
                        });
                    }
                    await storageServ?.replaceUser(updateUser);
                    users.value = users.value.map((user) => {
                        if (user.id === updateUser.id) {
                            return { ...updateUser };
                        } else {
                            return user;
                        }
                    });
                    centeredModalOpen.value = false;
                }
            };

            const handleUpdateUserActive = async (updUser: User) => {
                // Store the updated user's active field in the database
                if (db.value) {
                    const isConn = await sqliteServ?.isConnection(dbNameRef.value, false);
                    if (!isConn) {
                        const msg = 'Error handleUpdateUser: No DatabaseConnection';
                        console.error(msg);
                        Toast.show({
                            text: msg,
                            duration: 'long',
                        });
                    }
                    await storageServ?.updateUserActiveById(
                        updUser.id.toString(),
                        updUser.active
                    );
                    users.value = users.value.map((user: User) => {
                        if (user.id === updUser.id) {
                            // Clone the user and update the active property
                            return { ...user, active: updUser.active };
                        } else {
                            return user;
                        }
                    });
                }
            };

            const handleDeleteUser = async (userId: number) => {
                // Delete a user from the database by using the id field
                if (db.value) {
                    const isConn = await sqliteServ?.isConnection(dbNameRef.value, false);
                    if (!isConn) {
                        const msg = 'Error handleDeleteUser: No DatabaseConnection';
                        console.error(msg);
                        Toast.show({
                            text: msg,
                            duration: 'long',
                        });
                    }
                    await storageServ?.deleteUserById(userId.toString());
                    users.value = users.value.filter(
                    (user) => (user as User).id !== userId
                    );
                }
            };

            const handleEditUser = (user: User) => {
                // Get the user to update and open the modal edit form
                editUser.value = user;
                centeredModalOpen.value = true;
            };

            onMounted(() => {
                storageServ?.isInitCompleted.subscribe(async (value: boolean) => {
                    isInitComplete.value = value;
                    // Open the connection to the database
                    if (isInitComplete.value === true) {
                        if (platform === 'web') {
                            // Web Plaform
                            customElements
                            .whenDefined('jeep-sqlite')
                            .then(async () => {
                                await openDatabase();
                            })
                            .catch((error) => {
                                const msg = `Error open database: ${error}`;
                                console.log(msg);
                                Toast.show({
                                    text: msg,
                                    duration: 'long',
                                });
                            });
                        } else {
                            // Native Platforms
                            await openDatabase();
                        }
                    }
                });
            });

            onBeforeUnmount(() => {
            // Close the connection to the database
            sqliteServ
                ?.closeDatabase(dbNameRef.value, false)
                .then(() => {
                    isDatabase.value = false;
                })
                // eslint-disable-next-line @typescript-eslint/no-explicit-any
                .catch((error: any) => {
                    const msg = `Error close database:
                                ${error.message ? error.message : error}`;
                    console.error(msg);
                    Toast.show({
                        text: msg,
                        duration: 'long',
                    });
                });
            });

            watch(isDatabase, (newIsDatabase) => {
                if (newIsDatabase) {
                    // Get all users from the database
                    getAllUsers(db.value as SQLiteDBConnection)
                    // eslint-disable-next-line @typescript-eslint/no-explicit-any
                    .catch((error: any) => {
                        const msg = `close database:
                                    ${error.message ? error.message : error}`;
                        console.error(msg);
                        Toast.show({
                        text: msg,
                        duration: 'long',
                        });
                    });
                } else {
                    const msg = 'newDb is null';
                    console.error(msg);
                    Toast.show({
                    text: msg,
                    duration: 'long',
                    });
                }
            });

            return {
                centeredModalMarginTop,
                centeredModalOpen,
                dbInitialized,
                editUser,
                isInitComplete,
                users,
                handleAddUser,
                handleDeleteUser,
                handleEditUser,
                handleUpdateUser,
                handleUpdateUserActive,
            };
        },
    });
    </script>
    ```
 - Create a route for that page in the `routes.ts` file in the router folder.

 ```ts
    const routes: RouteRecordRaw[] = [
    {
        path: '/',
        component: () => import('layouts/MainLayout.vue'),
        children: [
            {
                path: '',
                name: 'home',
                component: () => import('pages/IndexPage.vue'),
                meta: { title: 'Quasar SQLite App' },
            },
            {
                path: '/users',
                name: 'users',
                component: () => import('pages/UsersPage.vue'),
                meta: { title: 'Managing Users' },
            },
        ],
    }, 
 ```

### Users Component Implementation

 In that tutorial we use a simple UI interface for CRUD operations through the use of a `UserForm` and a `UserList` components.
 
 - Under `src/components` folder create two new files `UserForm.vue` and `UserList.vue`.

 
 - Open the `components/UserForm.vue` file and copy the following code

    ```ts
    <template>
        <q-form @submit.prevent="submitForm">
            <!-- User Name input -->
            <q-input
            v-model="formData.username"
            label="Rose Miller"
            outlined
            dense
            :rules="[charsRule, twoWordsRule]"
            />
            <!-- User Email Version 2
            <q-input
            v-model="formData.email"
            label="Email"
            outlined
            dense
            :rules="[emailRule]"
            />
            -->
            <!-- Add more input fields as needed -->

            <q-btn v-if="isFormValid" type="submit" label="Submit" color="primary" />
        </q-form>
    </template>

    <script lang="ts">
    import { defineComponent, ref } from 'vue';
    import { User } from '../models/User';

    export default defineComponent({
        name: 'UserForm',
        props: {
            method: {
                type: String,
                required: true,
                validator: (value: string) => {
                    return ['add', 'update'].includes(value);
                },
            },
            user: {
                type: Object as () => User | null,
                default: null,
            },
            onAddUser: {
                type: Function,
                default: null,
            },
            onUpdateUser: {
                type: Function,
                default: null,
            },
        },
        setup(props) {
            const isFormValid = ref(false);
            const isFormInit = ref(false);
            const formData = ref({
                username: '',
                /* email: '', // version 2 */
            });
            if (props.method === 'update' && props.onUpdateUser && props.user) {
                formData.value.username = props.user.name;
                /* formData.value.email = props.user.email; // version 2 */
            }
            /* const emailRule = () => {
                if (isFormInit.value) return true;
                const isValid = /^[\w-]+(\.[\w-]+)*@([\w-]+\.)+[a-zA-Z]{2,7}$/.test(formData.value.email);
                return isValid || 'Invalid email format';
            } // version 2 */
            const charsRule = () => {
                if (isFormInit.value) return true;
                const isValid = /^[a-zA-Z0-9-' ']+$/.test(formData.value.username);
                return isValid || 'Only alphanumeric characters are allowed';
            };
            const twoWordsRule = () => {
                if (isFormInit.value) return true;
                const words = formData.value.username.trim().split(' ');
                const isValid = words.length >= 2 && words.every((word) => word !== '');
                return isValid || 'Must have at least 2 words';
            };
            const submitForm = () => {
                if (isFormValid.value) {
                    if (props.method === 'add' && props.onAddUser) {
                        const newUser = {
                            id: Date.now(), // do not care about the id value (generated by sqlite)
                            name: formData.value.username,
                            active: 1,
                            /* email: formData.value.email, // version 2 */
                        };
                        if (typeof props.onAddUser === 'function') {
                            props.onAddUser(newUser);
                        }
                    } else if (
                        props.method === 'update' &&
                        props.onUpdateUser &&
                        props.user
                    ) {
                        const updateUser = {
                            id: props.user.id,
                            name: formData.value.username,
                            active: props.user.active,
                            /* email: formData.value.email, // version 2 */
                        };
                        if (typeof props.onUpdateUser === 'function') {
                            props.onUpdateUser(updateUser);
                        }
                    }
                    formData.value.username = '';
                    /* formData.value.email = ''; // version 2 */
                    isFormInit.value = true;
                    isFormValid.value = false;
                }
            };
            return {
                formData,
                charsRule,
                twoWordsRule,
                submitForm,
                isFormValid,
                /* emailRule, // Version 2 */
            };
        },
    });
    </script>
    ``` 
 - Open the `components/UserList.vue` file and copy the following code

    ```ts
    <template>
        <q-list>
            <q-item v-for="user in users" :key="user.id">
            <q-item-section side middle>
                <q-checkbox
                :model-value="userActiveState(user)"
                dense
                @click="handleCheckboxClick(user)"
                />
            </q-item-section>
            <q-item-section>
                <q-item-label> {{ user.id }} - {{ user.name }} </q-item-label>
                <!-- <q-item-label> {{ user.email }} </q-item-label> // version 2 -->
            </q-item-section>
            <q-item-section side middle>
                <q-btn
                dense
                flat
                icon="edit"
                color="positive"
                @click="handleEditClick(user)"
                />
            </q-item-section>

            <!-- Delete Button with Trash Icon -->
            <q-item-section side middle>
                <q-btn
                dense
                flat
                icon="delete"
                color="negative"
                @click="handleDeleteClick(user.id)"
                />
            </q-item-section>
            </q-item>
        </q-list>
    </template>
    <script lang="ts">
    import { defineComponent } from 'vue';
    import { User } from '../models/User';

    export default defineComponent({
        name: 'UserList',
        props: {
            users: Array as () => User[],
            onUpdateUserActive: Function,
            onDeleteUser: Function,
            onEditUser: Function,
        },
        setup(props) {
            const userActiveState = (user: User) => {
            // Convert user.active to boolean
            return user.active === 1;
            };
            const handleCheckboxClick = (user: User) => {
                // Create a new user object with the updated active value
                const updatedUser = { ...user, active: user.active === 1 ? 0 : 1 };
                if (typeof props.onUpdateUserActive === 'function') {
                        props.onUpdateUserActive(updatedUser);
                }
            };
            const handleDeleteClick = (userId: number) => {
                if (typeof props.onDeleteUser === 'function') {
                    props.onDeleteUser(userId);
                }
            };
            const handleEditClick = (user: User) => {
                if (typeof props.onEditUser === 'function') {
                    props.onEditUser(user);
                }
            };

            return {
                userActiveState,
                handleCheckboxClick,
                handleDeleteClick,
                handleEditClick,
            };
        },
    });
    </script>
      ```

    !!! Caution Between the <q-item-label> </q-item-label tags
    You must read `Double left Curly Bracket` user.id `Double Right Curly Bracket` - `Double left Curly Bracket` user.name `Double Right Curly Bracket`




### Create the QuerySQLite Hook

The Querysqlite hook is a generic query to query date from a database.

 - Create a `hooks`folder under `src` folder.
 - Create a `useQuerySQLite.ts` file under the `src/hooks`folder and copy the below code in it.

    ```ts
    import { SQLiteDBConnection } from '@capacitor-community/sqlite';

    export const useQuerySQLite = async (
        db: SQLiteDBConnection,
        stmt: string,
        values: never[]
    ) => {
        if (db) {
            try {
                const queryResult = await db.query(stmt, values);
                return queryResult.values as never[];
            // eslint-disable-next-line @typescript-eslint/no-explicit-any
            } catch (err: any) {
                const msg = err.message ? err.message : err;
                throw new Error(`useQuerySQLite: ${msg}`);
            }
        }
        return [];
    };

    ```

### Update Config

 - For Web application only the appID of the `capacitor.config.ts` must be amended with `YOUR_APP_ID`


### Run the Web SQLite App

 - To run the Web app use the below command

    ```bash
    npm run dev
    ``` 
   
   When complete successfuly, the app will open in your default browser at

    ```
    http://localhost:9000/
    ```

 - This will bring you to the `Home` page. The video demonstrates the use of the application

 
    <video width="50%"  controls>
      <source src="/videos/Part-1-Quasar-SQLite-App.mov" type="video/mp4">
    </video><br>

### Video storyboard

    - <p>To open the menu, click on the <span style="font-size: 24px;"><ion-icon name="menu"></ion-icon></span> icon in the top right corner.</p>

    - <p> In the <strong><code>Menu Content</code></strong> click on <strong><code>Managing Users</code></strong></p>
 
    - <p> Enter a new user name (Sue Hellen) on the input field <strong><code>Rose Miller</code></strong></p>

    - <p> Click on the <strong><code>SUBMIT</code></strong> button</p>

    - <p><strong><code>Sue Hellen</code></strong>is now added to the Current Users list </p>

    - <p> Enter a new user name (Dave Wat) on the input field <strong><code>Rose Miller</code></strong></p>

    - <p> Click on the <strong><code>SUBMIT</code></strong> button</p>

    - <p><strong><code>Dave Wat</code></strong> is now added to the Current Users list </p>

    - <p> Update the <strong><code>active</code></strong> value for a user by pressing on the check icon on the left it will toggle the value from 1 to 0</p>

    - <p> Modify the name <strong><code>Dave Wat</code></strong> by clicking on the <span style="font-size: 24px;color: #21BA45;"><ion-icon name="pencil"></ion-icon></span> icon</p>

    - <p>The <strong><code>Update User</code></strong> modal form appears and update the name <strong><code>Dave Watson</code></strong></p>.

    - <p> Click on the <strong><code>SUBMIT</code></strong> button, the modal form disappears and the <strong><code>Current Users</code></strong> list is updated</p>

    - <p> Enter a new user name (James Dean) on the input field <strong><code>Rose Miller</code></strong></p>

    - <p> Click on the <strong><code>SUBMIT</code></strong> button</p>

    - <p><strong><code>James Dean</code></strong> is now added to the Current Users list </p>

    -<p> Delete the user <strong><code>James Dean</code></strong> by clicking on the <span style="font-size: 24px;color: #C10015;"><ion-icon name="trash"></ion-icon></span></p> icon

    -<p> Click on the <strong><code>Back</code></strong> button will route you back to the <strong><code>Home</code></strong> page </p> and save the database to store.

The database location can be see in the application tab of the development tools.

<div align="center"><br><img src="/images/Ionic7-Vue-SQLite-Database-Location.png" width="80%" /></div><br>

### Upgrade Database to Version 2

To upgrade the database to version 2, incremental upgrade statements will be used.
An `email` field is added to the users table in version 2.

The initial application code contains the code, but it has been accompanied by comments using the symbols '/*' and '*/'. These symbols require removal.

 - Open the file `user.statements.ts` in folder `src/upgrades` and remove the symbols.

 ```ts
    export const UserUpgradeStatements = [
        {
            toVersion: 1,
            statements: [
            `CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            active INTEGER DEFAULT 1
            );`,
            ],
        },
        /* add new statements below for next database version when required*/
        {
            toVersion: 2,
            statements: ['ALTER TABLE users ADD COLUMN email TEXT;'],
        },
    ];
 ```
 
 - The `user` model requires removal of symbols. Open the file `User.ts` in the `src/models` folder.

 ```ts
    export interface User {
        id: number;
        name: string;
        active: number;
        email: string; // Version 2
    }
 ```

 - The addition of an email input field affects the `UserForm` component. Open the file `UserForm.vue` in the folder `src/components`.

 ```ts
    <template>
        <q-form @submit.prevent="submitForm">
            <!-- User Name input -->
            <q-input
                v-model="formData.username"
                label="Rose Miller"
                outlined
                dense
                :rules="[charsRule, twoWordsRule]"
            />
            <!-- User Email Version 2 -->
            <q-input
                v-model="formData.email"
                label="Email"
                outlined
                dense
                :rules="[emailRule]"
            />
            <!-- Add more input fields as needed -->

            <q-btn v-if="isFormValid" type="submit" label="Submit" color="primary" />
        </q-form>
    </template>

    <script lang="ts">
    import { defineComponent, ref } from 'vue';
    import { User } from '../models/User';

    export default defineComponent({
        name: 'UserForm',
        props: {
            method: {
                type: String,
                required: true,
                validator: (value: string) => {
                    return ['add', 'update'].includes(value);
                },
            },
            user: {
                type: Object as () => User | null,
                default: null,
            },
            onAddUser: {
                type: Function,
                default: null,
            },
            onUpdateUser: {
                type: Function,
                default: null,
            },
        },
        setup(props) {
            const isFormValid = ref(false);
            const isFormInit = ref(false);
            const formData = ref({
                username: '',
                email: '', // version 2
            });
            if (props.method === 'update' && props.onUpdateUser && props.user) {
                formData.value.username = props.user.name;
                formData.value.email = props.user.email; // version 2
            }
            const emailRule = () => {
                if (isFormInit.value) return true;
                const isValid = /^[\w-]+(\.[\w-]+)*@([\w-]+\.)+[a-zA-Z]{2,7}$/.test(
                    formData.value.email
                );
                return isValid || 'Invalid email format';
            }; // version 2
            const charsRule = () => {
                if (isFormInit.value) return true;
                const isValid = /^[a-zA-Z0-9-' ']+$/.test(formData.value.username);
                return isValid || 'Only alphanumeric characters are allowed';
            };
            const twoWordsRule = () => {
                if (isFormInit.value) return true;
                const words = formData.value.username.trim().split(' ');
                const isValid = words.length >= 2 && words.every((word) => word !== '');
                return isValid || 'Must have at least 2 words';
            };
            const handleEnterKey = (event: { preventDefault: () => void }) => {
                if (isFormInit.value) {
                    isFormInit.value = false;
                    event.preventDefault();
                }
                // Check all rules and update isFormValid
                isFormValid.value =
                    charsRule() === true && twoWordsRule() === true && emailRule() === true; // version 2

                if (!isFormValid.value) {
                    // Prevent form submission on Enter key if the form is not valid
                    event.preventDefault();
                }
            };
            const submitForm = () => {
                if (isFormValid.value) {
                    if (props.method === 'add' && props.onAddUser) {
                        const newUser = {
                            id: Date.now(), // do not care about the id value (generated by sqlite)
                            name: formData.value.username,
                            active: 1,
                            email: formData.value.email, // version 2
                        };
                        if (typeof props.onAddUser === 'function') {
                            props.onAddUser(newUser);
                        }
                    } else if (
                    props.method === 'update' &&
                    props.onUpdateUser &&
                    props.user
                    )   {
                        const updateUser = {
                            id: props.user.id,
                            name: formData.value.username,
                            active: props.user.active,
                            email: formData.value.email, // version 2
                        };
                        if (typeof props.onUpdateUser === 'function') {
                            props.onUpdateUser(updateUser);
                        }
                    }
                    formData.value.username = '';
                    formData.value.email = ''; // version 2
                    isFormInit.value = true;
                    isFormValid.value = false;
                }
            };
            return {
                formData,
                charsRule,
                twoWordsRule,
                submitForm,
                isFormValid,
                emailRule, // Version 2
            };
        },
    });
    </script>
 ```

 - It also affects the `UserList` component as the email must be displayed. Open the file `UserList.vue` in the folder `src/components`.
 The `<!-- <q-item-label> {{ user.email }} </q-item-label> // version 2 -->`
 must be uncommented  like this `<q-item-label> {{ user.email }} </q-item-label>`.

 - run the application

    ```bash
    npm run dev
    ```
   When complete successfuly, the app will open in your default browser at

    ```
    http://localhost:9000/
    ```

 - This will bring you to the `Home` page. The video demonstrates the use of the application for the database version 2

 
    <video width="50%"  controls>
      <source src="/videos/Part-1-Quasar-SQLite-App2.mov" type="video/mp4">
    </video><br>

 - Video storyboard

    - <p>To open the menu, click on the <span style="font-size: 24px;"><ion-icon name="menu"></ion-icon></span> icon in the top right corner.</p>

    - <p> In the <strong><code>Menu Content</code></strong> click on <strong><code>Managing Users</code></strong></p>

    - <p> The <strong><code>email</code></strong> input field has been added to the <strong><code>Add New User</code></strong> form.</p>

    - <p> The <strong><code>Current Users</code></strong> list shows the existing users</p>

    - <p> Add the new user <strong><code>James Dean</code></strong> and its email <strong><code>james.dean@example.com</code></strong></p>.
    
    - <p> Click on the <strong><code>SUBMIT</code></strong> button. Check that the user list is updated.</p>

    - <p> Update the email for <strong><code>Sue Hellen</code></strong> user by clicking on the <span style="font-size: 24px;color: #21BA45;"><ion-icon name="pencil"></ion-icon></span> icon</p>

    - <p>The <strong><code>Update User</code></strong> modal form appears and update the email <strong><code>sue.hellen@example.com</code></strong></p>.

    - <p> Click on the <strong><code>SUBMIT</code></strong> button. Check that the user list is updated.</p>

    - <p> Update the email for <strong><code>Dave Watson</code></strong> user by clicking on the <span style="font-size: 24px;color: #21BA45;"><ion-icon name="pencil"></ion-icon></span> icon</p>

    - <p>The <strong><code>Update User</code></strong> modal form appears and update the email <strong><code>dave.watson@example.com</code></strong></p>.

    - <p> Click on the <strong><code>SUBMIT</code></strong> button. Check that the user list is updated.</p>

    - <p> The database <strong><code>Version 2</code></strong> is now fully updated.</p>

### Part 1 Conclusion

We have completed the Part 1 - Web application of the Quasar SQLite Database CRUD App Example Tutorial using Quasar Framework and @capacitor-community/sqlite.

We learned how to implement the '@capacitor-community/sqlite' plugin in the Quasar Framework on a Web platform.

We learned to use `provide/inject` to ensure that the services are unique and available to all components within the app.

We learned how to create an application menu.

We learned how to used some basic methods of the '@capacitor-community/sqlite' plugin to create a CRUD application from scratched and store persistent SQLite data into the Web IndexedDB browser database.

We learned how to upgrade the `version` of the database by using the incremental upgrade statements process.

Enjoy your development from there.


## Part 2 - Native - Table of Contents

---
 - [Update Capacitor Config file](#update-capacitor-config-file)
 - [Install Native Required Packages](#install-native-required-packages)
 - [Update Package.json Scripts](#update-packagejson-scripts)
 - [Run the iOS App](#run-the-ios-app)
 - [Run the Android App](#run-the-android-app)
 - [Run the Electron App](#run-the-electron-app)
 - [Part 2 Conclusion](#part-2-conclusion)

---

### Update Capacitor Config file

The `capacitor.config.ts` file specifies parameters that the plugin needs, such as database location, database encryption, and the use of biometric authentication for access security. Such settings may be platform specific.

CAUTION: Make sure that the `webDir` is set to `dist/spa`.

 - Please modify the `capacitor.config.ts` file as below :

    ```ts
    import { CapacitorConfig } from '@capacitor/cli';

    const config: CapacitorConfig = {
    appId: 'YOUR_APP_ID',
    appName: 'YOUR_APP_NAME',
    webDir: 'dist/spa',
    loggingBehavior: 'debug',
    server: {
        androidScheme: "http"
    },
    plugins: {
        CapacitorSQLite: {
        iosDatabaseLocation: 'Library/CapacitorDatabase',
        iosIsEncryption: false,
        iosKeychainPrefix: 'YOUR_APP_NAME',
        iosBiometric: {
            biometricAuth: false,
            biometricTitle : "Biometric login for capacitor sqlite"
        },
        androidIsEncryption: false,
        androidBiometric: {
            biometricAuth : false,
            biometricTitle : "Biometric login for capacitor sqlite",
            biometricSubTitle : "Log in using your biometric"
        },
        electronIsEncryption: false,
        electronWindowsLocation: "C:\\ProgramData\\CapacitorDatabases",
        electronMacLocation: "/Users/YOUR_NAME/CapacitorDatabases",
        electronLinuxLocation: "Databases"
        }
    }
    };
    export default config;
    ```

### Install Native Required Packages

 - In order to build Android, iOS, or Electron applications on specific devices, it is necessary to install the necessary capacitor packages first. So run the following commands:

    ```bash
    npm i @capacitor/android
    npm i @capacitor/ios
    npm i @capacitor-community/electron
    npm i -D rimraf
    ```
 
 - Integrate the various platforms into the application.

    ```bash
    npm run build
    npx cap add android
    npx cap add ios
    npx cap add @capacitor-community/electron
    ```

### Update Package.json Scripts

 - First install a new package to remove the sql.js wasm file to reduce the native package size

    ```bash
    npm install -D rimraf
    ```

 - Open the `package.json` file and replace the scripts section with:

    ```json
    "scripts": {
        "lint": "eslint --ext .js,.ts,.vue ./",
        "format": "prettier --write \"**/*.{js,ts,vue,css,html,md,json}\" --ignore-path .gitignore",
        "test": "echo \"No test specified\" && exit 0",
        "dev": "npm run copy:sql:wasm && quasar dev",
        "build": "quasar build",
        "build:web": "npm run copy:sql:wasm && quasar build",
        "build:native": "npm run remove:sql:wasm && quasar build",
        "copy:sql:wasm": "copyfiles -u 3 node_modules/sql.js/dist/sql-wasm.wasm public/assets",
        "remove:sql:wasm": "rimraf public/assets/sql-wasm.wasm",
        "ios:start": "npm run build:native && npx cap sync && npx cap copy && npx cap open ios",
        "android:start": "npm run build:native && npx cap sync && npx cap copy && npx cap open android",
        "electron:install": "cd electron && npm install && cd ..",
        "electron:prepare": "npm run build:native && npx cap sync @capacitor-community/electron && npx cap copy @capacitor-community/electron",
        "electron:start": "npm run electron:prepare && cd electron && npm run electron:start"
    },
    ```

### Run the iOS App

 - Before running on iOS, the code has to be modified to account for the difference between the `Super View` and the `Safe Area`.

    - Open the `app.css` file under the `src/css` folder and add:

    ```css
    .ios {
        top: 44px;
    }
    ```

    - Change the code in file `src/pages/IndexPage.vue> to match this one.

    ```ts
    <template>
        <q-page>
            <app-logo></app-logo>
            <app-intro-text></app-intro-text>
        </q-page>
    </template>

    <script lang="ts">
    import { defineComponent, onMounted, inject } from 'vue';
    import AppLogo from 'components/AppLogo.vue';
    import AppIntroText from 'components/AppIntroText.vue';
    import SQLiteService from 'src/services/sqliteService';

    export default defineComponent({
    name: 'IndexPage',
    components: { AppLogo, AppIntroText },
        setup() {
            // Inject services
            const sqliteServ: SQLiteService | undefined = inject('sqliteServ');
            const platform = sqliteServ?.getPlatform();
            onMounted(() => {
                if (platform === 'ios') {
                    const headerEl = document.querySelector('.q-header');
                    headerEl?.classList.add('ios');
                    const pageEl = document.querySelector('.q-page');
                    pageEl?.classList.add('ios');
                }
            });
            return {};
        },
    });
    </script>
    ```
    - Change the unMounted method code in file `src/pages/UsersPage.vue> to match this one.

    ```ts 
    onMounted(() => {
      if (platform === 'ios') {
        const headerEl = document.querySelector('.q-header');
        headerEl?.classList.add('ios');
        const pageEl = document.querySelector('.q-page');
        pageEl?.classList.add('ios');
      }
      storageServ?.isInitCompleted.subscribe(async (value: boolean) => {
        isInitComplete.value = value;
        // Open the connection to the database
        if (isInitComplete.value === true) {
          if (platform === 'web') {
            // Web Plaform
            customElements
              .whenDefined('jeep-sqlite')
              .then(async () => {
                await openDatabase();
              })
              .catch((error) => {
                const msg = `Error open database: ${error}`;
                console.log(msg);
                Toast.show({
                  text: msg,
                  duration: 'long',
                });
              });
          } else {
            // Native Platforms
            await openDatabase();
          }
        }
      });
    });
    ```
 - Open the file `src/layouts/MainLayout.vue> to match this one.

    - add in the `import` section

    ```ts
    import { Capacitor } from '@capacitor/core';
    ```
 
    - modify the `setup` method 

    ```ts
    setup() {
        const rightDrawerOpen = ref(false);
        const platform = Capacitor.getPlatform();
        return {
            pages: pageList,
            rightDrawerOpen,
            toggleRightDrawer() {
                rightDrawerOpen.value = !rightDrawerOpen.value;
                if (platform === 'ios') {
                    // eslint-disable-next-line @typescript-eslint/no-explicit-any
                    const menuEl: any = document.querySelector('.q-drawer');
                    menuEl.style.top = '44px';
                }
            },
        };
    },


 - Run the following command:

    ```bash
    npm run ios:start
    ```

 - In Xcode wait for indexed file to complete, clean the project and run the app.

 - This will bring you to the `Home` page. The video demonstrates the use of the application for the database version 1

 
    <video width="50%"  controls>
      <source src="/videos/Part-2-iOS-Quasar-SQLite-App.mov" type="video/mp4">
    </video><br>

 - The `video storyboard` is the same that for the Web Part1 [Video Storyboard](#video-storyboard).

### Run the Android App

 - Run the following command:

    ```bash
    npm run android:start
    ```

 - In Android Studio

    - go to the menu Android/Preferences/Build, Execution, Deployment/Build Tools/Gradle and then select the gradle JDK `17 Oracle OpenJDK Version 17.0.7`. Then press `Apply` and `OK`.
    - go to File/Sync Project With Gradle Files.
    - go to Build/Clean Project.
    - select your Emulator or Physical Device.
    - run the app
 
 - This will bring you to the `Home` page. The video demonstrates the use of the application for the database version 1

 
    <video width="50%"  controls>
      <source src="/videos/Part-2-Android-Quasar-SQLite-App.mov" type="video/mp4">
    </video><br>

 - The `video storyboard` is the same that for the Web Part1 [Video Storyboard](#video-storyboard).

### Run the Electron App

 - Run the following command:

    ```bash
    cd electron
    ```
 - Open the `package.json` file and replace the scripts:, dependencies:, devDependencies: sections with:

    ```json
    "scripts": {
        "build": "tsc && electron-rebuild",
        "electron:start-live": "node ./live-runner.js",
        "electron:start": "npm run build &&  electron --inspect=5858 ./",
        "electron:pack": "npm run build && electron-builder build --dir -c ./electron-builder.config.json",
        "electron:make": "npm run build && electron-builder build -c ./electron-builder.config.json -p always"
    },
    "dependencies": {
        "@capacitor-community/electron": "^5.0.1",
        "@capacitor-community/sqlite": "^5.4.2-2",
        "better-sqlite3-multiple-ciphers": "^9.1.1",
        "chokidar": "~3.5.3",
        "crypto": "^1.0.1",
        "crypto-js": "^4.1.1",
        "electron-is-dev": "~2.0.0",
        "electron-json-storage": "^4.6.0",
        "electron-serve": "~1.2.0",
        "electron-unhandled": "~4.0.1",
        "electron-updater": "^6.1.7",
        "electron-window-state": "^5.0.3",
        "jszip": "^3.10.1",
        "node-fetch": "2.6.7"
    },
    "devDependencies": {
        "@electron/rebuild": "^3.4.0",
        "@types/better-sqlite3": "^7.6.8",
        "@types/crypto-js": "^4.1.1",
        "@types/electron-json-storage": "^4.5.0",
        "electron": "^27.1.3",
        "electron-builder": "^24.9.1",
        "typescript": "^5.0.4"
    },

    ```

 - In your Editor delete the `node_modules` folder and `package-lock.json` file.

 - Then run the following commands:

    ```bash
    npm install
    cd ..
    npm run electron:start
    ```

 - This will bring you to the `Home` page. The video demonstrates the use of the application for the database version 1

 
    <video width="50%"  controls>
      <source src="/videos/Part-2-Electron-Quasar-SQLite-App.mov" type="video/mp4">
    </video><br>

 - The `video storyboard` is the same that for the Web Part1 [Video Storyboard](#video-storyboard).


### Part 2 Conclusion

We have completed the Part 2 - Native and Electron applications of the Quasar SQLite Database CRUD App Example Tutorial using Quasar Framework and @capacitor-community/sqlite.

We have learned to adapt the `capacitor.config.ts`file and the modify the scripts: section of the `package.json`file for native apps

We learned how to modify the code to adapt to the `safe area`on iOS device.

We learned how to build and run the application for each native platforms.

Enjoy your development from there.

