
# Ionic 7 SQLite Database CRUD App Example Tutorial using Vue3 and @capacitor-community/sqlite
---

*last updated on September 7, 2023 by Quéau Jean Pierre*

In that tutorial we will learned how to create a Ionic7/Vue basic CRUD application and implement the @capacitor-community/sqlite plugin to store the data in a SQLite database.

The first part of the tutorial will concentrate on how to create that application and run it on a Web browser where the data will be stored on an Indexed database using sql.js and localForage modules.
Go to [Part 1 - Web - Table of Contents](#part-1---web---table-of-contents)

The application can be found at [Part-1/ionic7-vue-sqlite-app](https://github.com/jepiqueau/blog-tutorials-apps/tree/main/Part-1/ionic7-vue-sqlite-app)


Thanks to the Ionic Team and their hard work to bring CAPACITOR 5, the second part will concentrate on native platforms (iOS and Android) and also on Electron platform.
Go to [Part 2 - Native - Table of Contents](#part-2---native---table-of-contents)

The application can be found at [Part-2/ionic7-vue-sqlite-app](https://github.com/jepiqueau/blog-tutorials-apps/tree/main/Part-2/ionic7-vue-sqlite-app)



## Part 1 - Web - Table of Contents

---
 - [Install New Ionic Application](#install-new-ionic-application)
 - [Install Required Packages](#install-required-packages)
 - [Create the SQLite Database](#create-the-sqlite-database)
 - [Create CRUD and SQLite Services](#create-crud-and-sqlite-services)
 - [Vue Plugin Implementation](#vue-plugin-implementation)
 - [Users Page Implementation](#users-page-implementation)
 - [Users Component Implementation](#users-component-implementation)
 - [Modify the Home Page](#modify-the-home-page)
 - [Add a Menu to the App](#add-a-menu-to-the-app)
 - [Update Config and Package.json Scripts](#update-config-and-packagejson-scripts)
 - [Run the Web SQLite App](#run-the-web-sqlite-app)
 - [Part 1 Conclusion](#part-1-conclusion)

---

### Install New Ionic Application

 - Install the latest version of Ionic CLI globally installed on your device, run the below command.

    ```bash
    sudo npm install -g @ionic/cli
    ```

 - Create a new blank Ionic Vue app

    ```bash
    ionic start ionic7-vue-sqlite-app blank --type=vue --capacitor
    ```

 - Go inside the project folder

    ```bash
    cd ./ionic7-vue-sqlite-app
    ```

### Install Required Packages

 - run the below commands

    ```bash
    npm install --save @capacitor-community/sqlite
    npm install --save @capacitor/toast
    npm install --save @ionic/pwa-elements
    npm install --save-dev copyfiles
    ```

### Create the SQLite Database

 - To easily upgrade the database from one version to the next, the database is created through an upgrade statement. In your editor, create a folder `upgrades` under the `src` folder and then create a new file `upgrades/user.upgrade.statements.ts`.

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
        id: number
        name: string
        active: number
        /* for version 2
        email: string
        */
    }
    ```

### Create CRUD and SQLite Services

 - To create CRUD and SQLite services, in your file editor, add a new `src/services` folder and create the two following files:

    - `services/sqliteService.ts`
    - `services/storageService.ts`

 - Open the `services/sqliteService.ts` file and replace with the following code.

    ```ts
    import { CapacitorSQLite, SQLiteConnection, SQLiteDBConnection, capSQLiteUpgradeOptions } from '@capacitor-community/sqlite';
    import { Capacitor } from '@capacitor/core';

    export interface ISQLiteService {
        getPlatform(): string
        initWebStore(): Promise<void>
        addUpgradeStatement(options: capSQLiteUpgradeOptions): Promise<void> 
        openDatabase(dbName: string, loadToVersion: number, readOnly: boolean): Promise<SQLiteDBConnection> 
        closeDatabase(dbName: string, readOnly: boolean): Promise<void>
        saveToStore(dbName: string): Promise<void>
        saveToLocalDisk(dbName: string): Promise<void>
        isConnection(dbName: string, readOnly: boolean): Promise<boolean>
    };

    class SQLiteService implements ISQLiteService {
        platform = Capacitor.getPlatform();
        sqlitePlugin = CapacitorSQLite;
        sqliteConnection = new SQLiteConnection(CapacitorSQLite);
        dbNameVersionDict: Map<string, number> = new Map();

        getPlatform(): string {
            return this.platform;
        }
        async initWebStore() : Promise<void>  {
            try {
                await this.sqliteConnection.initWebStore();
            } catch(error: any) {
                const msg = error.message ? error.message : error;
                throw new Error(`sqliteService.initWebStore: ${msg}`);
            }
            return;
        }
        async addUpgradeStatement(options: capSQLiteUpgradeOptions): Promise<void> {
            try {
                await this.sqlitePlugin.addUpgradeStatement(options);
            } catch(error: any) {
                const msg = error.message ? error.message : error;
                throw new Error(`sqliteService.addUpgradeStatement: ${msg}`);
            }
            return;
        }
        async openDatabase(dbName:string, loadToVersion: number,
                    readOnly: boolean): Promise<SQLiteDBConnection>  {
            this.dbNameVersionDict.set(dbName, loadToVersion);
            let encrypted = false;
            const mode = encrypted ? "secret" : "no-encryption";
            try {
                let db: SQLiteDBConnection;
                const retCC = (await this.sqliteConnection.checkConnectionsConsistency()).result;
                let isConn = (await this.sqliteConnection.isConnection(dbName, readOnly)).result;
                if(retCC && isConn) {
                db = await this.sqliteConnection.retrieveConnection(dbName, readOnly);
                } else {
                    db = await this.sqliteConnection
                            .createConnection(dbName, encrypted, mode, loadToVersion, readOnly);
                }
                const jeepSQlEL = document.querySelector("jeep-sqlite")
        
                await db.open();
                const res = await db.isDBOpen();
                return db;
            
            } catch(error: any) {
                const msg = error.message ? error.message : error;
                throw new Error(`sqliteService.openDatabase: ${msg}`);
            }

        }
        async isConnection(dbName:string, readOnly: boolean): Promise<boolean> {
            try {
                const isConn = (await this.sqliteConnection.isConnection(dbName, readOnly)).result;
                if (isConn != undefined) {
                    return isConn
                } else {
                    throw new Error(`sqliteService.isConnection undefined`);
                }
            
            } catch(error: any) {
                const msg = error.message ? error.message : error;
                throw new Error(`sqliteService.isConnection: ${msg}`);
            }
        }
        async closeDatabase(dbName:string, readOnly: boolean):Promise<void> {
            try {
                const isConn = (await this.sqliteConnection.isConnection(dbName, readOnly)).result;
                if(isConn) {
                    await this.sqliteConnection.closeConnection(dbName, readOnly);
                }
                return;
            } catch(error: any) {
                const msg = error.message ? error.message : error;
                throw new Error(`sqliteService.closeDatabase: ${msg}`);
            }
        }
        async saveToStore(dbName: string): Promise<void> {
            try {
                await this.sqliteConnection.saveToStore(dbName);
                return;
            } catch(error: any) {
                const msg = error.message ? error.message : error;
                throw new Error(`sqliteService.saveToStore: ${msg}`);
            }
        }
        async saveToLocalDisk(dbName: string): Promise<void> {
            try {
                await this.sqliteConnection.saveToLocalDisk(dbName);
                return;
            } catch(err:any) {
                const msg = err.message ? err.message : err;
                throw new Error(`sqliteService.saveToLocalDisk: ${msg}`);
            }
        }
    }
    export default new SQLiteService();
    ```

 - Open the `services/storage.service.ts` and replace the code with

    ```ts
    import {platform} from '../App';
    import { BehaviorSubject } from 'rxjs';
    import {ISQLiteService } from '../services/sqliteService'; 
    import {IDbVersionService } from '../services/dbVersionService';
    import { SQLiteDBConnection } from '@capacitor-community/sqlite';
    import { UserUpgradeStatements } from '../upgrades/user.upgrade.statements';
    import { User } from '../models/User';

    export interface IStorageService {
        initializeDatabase(): Promise<void>
        getUsers(): Promise<User[]>
        addUser(user: User): Promise<number>
        updateUserById(id: string, active: number): Promise<void> 
        deleteUserById(id: string): Promise<void>
        getDatabaseName(): string
        getDatabaseVersion(): number
    };
    class StorageService implements IStorageService  {
        versionUpgrades = UserUpgradeStatements;
        loadToVersion = UserUpgradeStatements[UserUpgradeStatements.length-1].toVersion;
        db!: SQLiteDBConnection;
        database: string = 'myuserdb';
        sqliteServ!: ISQLiteService;
        dbVerServ!: IDbVersionService;
        isInitCompleted = new BehaviorSubject(false);

        constructor(sqliteService: ISQLiteService, dbVersionService: IDbVersionService) {
            this.sqliteServ = sqliteService;
            this.dbVerServ = dbVersionService;
        }
        
        getDatabaseName(): string {
            return this.database;
        }
        getDatabaseVersion(): number {
            return this.loadToVersion;
        }
        async initializeDatabase(): Promise<void> {
            // create upgrade statements
            try {
                await this.sqliteServ.addUpgradeStatement({database: this.database,
                                                    upgrade: this.versionUpgrades});
                this.db = await this.sqliteServ.openDatabase(this.database, this.loadToVersion, false);
                const isData = await this.db.query("select * from sqlite_sequence");
                if(isData.values!.length === 0) {
                // create database initial users if any

                }

                this.dbVerServ.setDbVersion(this.database,this.loadToVersion);
                if( platform === 'web') {
                await this.sqliteServ.saveToStore(this.database);
                }
                this.isInitCompleted.next(true);
            } catch(error: any) {
                const msg = error.message ? error.message : error;
                throw new Error(`storageService.initializeDatabase: ${msg}`);
            }
        }
        async getUsers(): Promise<User[]>  {
            return (await this.db.query('SELECT * FROM users;')).values as User[];
        }
        async addUser(user: User): Promise<number> {
            const sql = `INSERT INTO users (name) VALUES (?);`;
            const res = await this.db.run(sql,[user.name]);
            if (res.changes !== undefined
                && res.changes.lastId !== undefined && res.changes.lastId > 0) {
                return res.changes.lastId;
            } else {
                throw new Error(`storageService.addUser: lastId not returned`);
            }
        }
        async updateUserById(id: string, active: number): Promise<void> {
            const sql = `UPDATE users SET active=${active} WHERE id=${id}`;
            await this.db.run(sql);
        }
        async deleteUserById(id: string): Promise<void> {
            const sql = `DELETE FROM users WHERE id=${id}`;
            await this.db.run(sql);
        }

    }
    export default StorageService;
    ```    

 To manage the database versionning, a new service `DbnameVersionService` is required. In your file editor, go to `src/services` folder and create the `dbVersionService.ts` file


 - Open the `services/dbVersionService.ts` file and replace the code with

    ```ts
    export interface IDbVersionService {
        setDbVersion(dbName: string, version: number): void
        getDbVersion(dbName: string):number| undefined
    };
    class DbVersionService implements IDbVersionService  {
        dbNameVersionDict: Map<string, number> = new Map();

        setDbVersion(dbName: string, version: number) {
            this.dbNameVersionDict.set(dbName, version);
        };
        getDbVersion(dbName: string): number | undefined {
            const version =  this.dbNameVersionDict.get(dbName);
            return version;
        };
    }
    export default new DbVersionService();
    ```

### Vue Plugin Implementation

 For Web applications, the plugin uses the Stencil component `jeep-sqlite` which implies some Vue files modification

 - Open the `main.ts` file and replace the code with the following

    ```ts
    import { createApp } from 'vue'
    import App from './App.vue'
    import router from './router';

    import { IonicVue } from '@ionic/vue';

    /* Core CSS required for Ionic components to work properly */
    import '@ionic/vue/css/core.css';

    /* Basic CSS for apps built with Ionic */
    import '@ionic/vue/css/normalize.css';
    import '@ionic/vue/css/structure.css';
    import '@ionic/vue/css/typography.css';

    /* Optional CSS utils that can be commented out */
    import '@ionic/vue/css/padding.css';
    import '@ionic/vue/css/float-elements.css';
    import '@ionic/vue/css/text-alignment.css';
    import '@ionic/vue/css/text-transformation.css';
    import '@ionic/vue/css/flex-utils.css';
    import '@ionic/vue/css/display.css';

    /* Theme variables */
    import './theme/variables.css';

    import { Capacitor } from '@capacitor/core';
    import { JeepSqlite } from 'jeep-sqlite/dist/components/jeep-sqlite';
    import { defineCustomElements as pwaElements} from '@ionic/pwa-elements/loader';
    import SqliteService from './services/sqliteService'; 
    import DbVersionService from './services/dbVersionService';
    import StorageService from './services/storageService';
    import InitializeAppService from './services/initializeAppService';

    pwaElements(window);
    customElements.define('jeep-sqlite', JeepSqlite);
    const platform = Capacitor.getPlatform();

    const app = createApp(App)
    .use(IonicVue)
    .use(router);


    // Set the platform as global properties on the app
    app.config.globalProperties.$platform = platform;

    // Define and instantiate the required services
    const sqliteServ = new SqliteService();
    const dbVersionServ = new DbVersionService();
    const storageServ = new StorageService(sqliteServ, dbVersionServ);
    // Set the services as global properties on the app
    app.config.globalProperties.$sqliteServ = sqliteServ;
    app.config.globalProperties.$dbVersionServ = dbVersionServ;
    app.config.globalProperties.$storageServ = storageServ;

    //Define and instantiate the InitializeAppService
    const initAppServ = new InitializeAppService(sqliteServ, storageServ);
 
    const mountApp = () => {
        initAppServ.initializeApp()
        .then(() => {
            router.isReady().then(() => {
            app.mount('#app');
            });
        })
        .catch((error) => {
            console.error('App Initialization error:', error);
        });
    }

    if (platform !== "web") {
        mountApp();
    } else {
        window.addEventListener('DOMContentLoaded', async () => {
            const jeepEl = document.createElement("jeep-sqlite");
            document.body.appendChild(jeepEl);
            customElements.whenDefined('jeep-sqlite').then(() => {
                mountApp();
            })
            .catch ((err) => {
                console.error('jeep-sqlite creation error:', err);
            });
        });
    }
    ``` 
 As the goal is to ensure that the services (`SqliteService`, `DbVersionService` and `StorageService`) are unique and properly shared and available throughout the components, they have been set as global properties on the app.

 - To initialize the application, an `InitializeAppService` service is execute prior mounting the app and has to be created.

 - In your Editor add a new file `initializeAppService.ts` file under the folder `src/services`.

 - In your editor, open the file `services/initializeAppService.ts` and replace the code with:

    ```ts
    import {ISQLiteService } from '../services/sqliteService'; 
    import {IStorageService } from '../services/storageService'; 

    export interface IInitializeAppService {
        initializeApp(): Promise<boolean>
    };

    class InitializeAppService implements IInitializeAppService  {
        appInit = false;
        sqliteServ!: ISQLiteService;
        storageServ!: IStorageService;
        platform!: string;

        constructor(sqliteService: ISQLiteService, storageService: IStorageService) {
            this.sqliteServ = sqliteService;
            this.storageServ = storageService;
            this.platform = this.sqliteServ.getPlatform();
        }
        async initializeApp() : Promise<boolean> {
            if(!this.appInit) {
                try {
                    if (this.platform === 'web') {
                        await this.sqliteServ.initWebStore();
                    }
                    // Initialize the myuserdb database
                    await this.storageServ.initializeDatabase();
                    if( this.platform === 'web') {
                        await this.sqliteServ.saveToStore(this.storageServ.getDatabaseName());
                    }
                    this.appInit = true;
                } catch(error: any) {
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
    <ion-app>
        <ion-router-outlet />
    </ion-app>
    </template>

    <script setup lang="ts">
    import { IonApp, IonRouterOutlet } from '@ionic/vue';
    import AppMenu from '@/components/AppMenu.vue';
    </script>
    ``` 

 - Add a route `/users` in the `src/router/index.ts` file by adding

  ```ts
  import { createRouter, createWebHistory } from '@ionic/vue-router';
    import { RouteRecordRaw } from 'vue-router';
    import HomePage from '../views/HomePage.vue';
    import UsersPage from '../views/UsersPage.vue';

    const routes: Array<RouteRecordRaw> = [
        {
            path: '/',
            redirect: '/home'
        },
        {
            path: '/home',
            name: 'Home',
            component: HomePage
        },
        {
            path: "/users",
            name: "Users",
            component: UsersPage,
        },
    ]

    const router = createRouter({
        history: createWebHistory(import.meta.env.BASE_URL),
        routes
    })

    export default router
  ```

### Users Page Implementation

 As you can see above, a route `/users` to a `UsersPage` has been added.
 Let's create it.

 - In your Editor, under `src/views` folder add a new page `UsersPage.vue`.

 - Then open the `UsersPage.vue` file and add the following code:

    ```ts
    <template>
        <ion-page>
        <ion-header>
            <ion-toolbar>
            <ion-title>Managing Users</ion-title>
            <ion-buttons slot="start">
                <ion-back-button text="home" default-href="/home"></ion-back-button>
            </ion-buttons>
            </ion-toolbar>
        </ion-header>
        <ion-content>
            <div v-if="isInitComplete && dbInitialized">
            <ion-card>
                <h1>Add New User</h1>
                <user-form :onAddUser="handleAddUser" />
            </ion-card>
            <ion-card>
                <h2>Current Users</h2>
                <user-list :users="users" :onUpdateUser="handleUpdateUser" :onDeleteUser="handleDeleteUser" />
            </ion-card>
            </div>
        </ion-content>
        </ion-page>
    </template>
    <script lang="ts">
    import { defineComponent, ref, computed, getCurrentInstance, onMounted,
            onBeforeUnmount, watch, Ref } from 'vue';
    import { useIonRouter } from '@ionic/vue';
    import { IonPage, IonHeader, IonToolbar, IonButtons, IonBackButton, IonTitle,
            IonContent, IonCard} from '@ionic/vue';
    import { Toast } from '@capacitor/toast';
    import { User } from '@/models/User';
    import UserForm from '@/components/UserForm.vue';
    import UserList from '@/components/UserList.vue';
    import { useQuerySQLite } from '@/hooks/UseQuerySQLite';
    import { SQLiteDBConnection } from '@capacitor-community/sqlite';

    export default defineComponent({
        name: 'TestPage',
        components: { IonPage, IonHeader, IonToolbar, IonButtons,
         IonBackButton, IonTitle,
         IonContent, IonCard, UserForm, UserList },
        setup() {
            const dbNameRef = ref('');
            const isInitComplete = ref(false);
            const isDatabase = ref(false);
            const users = ref<User[]>([]);
            const db = ref(null);
            const appInstance = getCurrentInstance();

            const router = useIonRouter();
        
            const sqliteServ = appInstance?.appContext.config.globalProperties.$sqliteServ;
            const storageServ = appInstance?.appContext.config.globalProperties.$storageServ;

            const dbInitialized = computed(() => !!db.value);
            const platform = sqliteServ.getPlatform();

            const getAllUsers = async (db: Ref<SQLiteDBConnection|null>) => {
                const stmt = 'SELECT * FROM users';
                const values: any[] = [];
                const fetchData = await useQuerySQLite(db, stmt, values);
                users.value = fetchData;         
            }

            const openDatabase = async () => {
                try {
                    const dbUsersName = storageServ.getDatabaseName();
                    dbNameRef.value = dbUsersName;
                    const version = storageServ.getDatabaseVersion();
            
                    const database = await sqliteServ.openDatabase(dbUsersName, version, false);
                    db.value = database;
                    isDatabase.value = true;
                } catch (error) {
                    const msg = `Error open database: ${error}`;
                    console.error(msg);
                    Toast.show({
                        text: msg,
                        duration: 'long'
                    });
                }
            };
            const handleAddUser = async (newUser: User) => {
                if (db.value) {
                    const isConn = await sqliteServ.isConnection(dbNameRef.value, false);
                    const lastId = await storageServ.addUser(newUser);
                    newUser.id = lastId;
                    users.value.push(newUser as never);
                }
            };
            const handleUpdateUser = async (updUser: User) => {
                if (db.value) {
                    const isConn = await sqliteServ.isConnection(dbNameRef.value, false);
                    await storageServ.updateUserById(updUser.id.toString(), updUser.active);
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
                if (db.value) {
                    const isConn = await sqliteServ.isConnection(dbNameRef.value, false);
                    await storageServ.deleteUserById(userId.toString());
                    users.value = users.value.filter(user => (user as User).id !== userId);
                }
            };
            onMounted(() => {
                const initSubscription = storageServ.isInitCompleted.subscribe(async (value: boolean) => {
                    isInitComplete.value = value;
                    if (isInitComplete.value === true) {
                        const dbUsersName = storageServ.getDatabaseName();
                        if (platform === "web") {
                        customElements.whenDefined('jeep-sqlite').then(async () => {
                            await openDatabase();
                        }).catch((error) => {
                            const msg = `Error open database: ${error}`;
                            console.log(msg);
                            Toast.show({
                            text: msg,
                            duration: 'long'
                            });
                        });
                        } else {
                        await openDatabase();
                        }
                    }
                });
            });
            onBeforeUnmount(() => {
                sqliteServ.closeDatabase(dbNameRef.value, false)
                .then(() => {
                    isDatabase.value = false;
                }).catch((error: any) => {
                    const msg = `Error close database:
                                ${error.message ? error.message : error}`;
                    console.error(msg);
                    Toast.show({
                    text: msg,
                    duration: 'long'
                    });
                });
            });

            watch(isDatabase, (newIsDatabase) => {
                if (newIsDatabase) {
                getAllUsers(db).then(() => {

                })
                .catch((error: any) => {
                    const msg = `close database:
                                ${error.message ? error.message : error}`;
                    console.error(msg);
                    Toast.show({
                    text: msg,
                    duration: 'long'
                    });
                });         
                } else {
                    const msg = `newDb is null`;
                    console.error(msg);
                    Toast.show({
                        text: msg,
                        duration: 'long'
                        });
                }
            });

            return {isInitComplete, dbInitialized, users, handleAddUser,
                     handleUpdateUser, handleDeleteUser}
            },
        });
    </script>
    ```

### Users Component Implementation

 In that tutorial we use a simple UI interface for CRUD operations through the use of a `UserForm` and a `UserList` Vue component.
 
 - Under `src/components` folder create two new files `UserForm.vue` and `UserList.vue`.

 
 - Open the `components/UserForm.vue` file and copy the following code

    ```ts
    <template>
        <div class="UserForm">
        <ion-input
            ref="ionNameEl"
            :value="nameModel"
            @ionInput="handleNameInput($event)"
            type="text"
            placeholder="Rose Miller"
        ></ion-input>
        <!-- version 2
        <ion-input
            ref="ionEmailEl"
            :value="emailModel"
            @ionInput="handleEmailInput($event)"
            type="email"
            placeholder="Email"
        ></ion-input>
        -->
        <ion-button expand="full" @click="handleSubmit" :disabled="!isFormValid">
            Add User
        </ion-button>
        </div>
    </template>
  
    <script lang="ts">
    import { ref } from 'vue';
    import { defineComponent, watch } from 'vue';
    import { IonInput, IonButton } from '@ionic/vue';
    
    export default defineComponent({
        name: 'UserForm',
        props: {
        onAddUser: Function,
        },
        components: {IonInput, IonButton},
        setup(props) {
            const ionNameEl = ref();
            const nameModel = ref('');
            const ionEmailEl = ref();
            const emailModel = ref('');
            const isFormValid = ref(false);
    
            const handleNameInput = (ev: { target: any; }) => {
                const value = ev.target!.value;
                const fileterdValue = value.replace(/[^a-zA-Z0-9-' ']+/g, '');
                nameModel.value = value;
        
                const inputCmp = ionNameEl.value;
                if (inputCmp !== undefined) {
                    inputCmp.$el.value  = fileterdValue;
                }
            };
            /* Version 2
            const handleEmailInput = (ev:  { target: any; }) => {
                const value = ev.target!.value;

                // Update both the state variable and
                // the component to keep them in sync.

                emailModel.value = value;

                const inputCmp = ionEmailEl.value;
                if (inputCmp !== null) {
                    inputCmp.$el.value = value;
                }

            };
            */    
            const handleSubmit = () => {
                if (nameModel.value /*&& emailModel.value */) {
                    const newUser = {
                        id: Date.now(), // do not care about the id value (generated by sqlite)
                        name: nameModel.value,
                        active: 1,
                        /* email: emailModel.value, // version 2 */
                    };
                    if (typeof props.onAddUser === 'function') {
                        props.onAddUser(newUser);
                    }
            
                    nameModel.value = '';
                    /* emailModel.value = ''; // version 2 */
                    isFormValid.value = false;
                }
            };
  
            const hasTwoWords = (input: string) => {
                const words = input.trim().split(' ');
                return words.length === 2 && words.every((word) => word !== '');
            };

            /* Version 2
            const isValidEmail = (email) => {
                if (email.length === 0) {
                    return false;
                }
                const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{1,}$/;
                return emailRegex.test(email);
            };
            */
            // Watch 'name' and 'email' for changes and update 'isFormValid'
            watch([nameModel/*, emailModel*/], ([newName/*, newEmail*/]) => {
                isFormValid.value = hasTwoWords(newName) /* && isValidEmail(newEmail)*/;
            });  
  
            return {
                ionNameEl,
                nameModel,
                /* emailModel, */
                isFormValid,
                handleNameInput,
                /*handleEmailInput,*/
                handleSubmit,
            };
        },
    });
    </script>
    ``` 
 - Open the `components/UserList.vue` file and copy the following code

    ```ts
    <template>
        <ion-list class="UserList">
        <ion-item v-for="user in users" :key="user.id">
            <ion-checkbox
            ref="ionActiveEl"
            slot="start"
            :checked="user.active==1 ? true: false"
            :activeModel="user.active"
            @ionChange="handleCheckboxChange(user)"
            ></ion-checkbox>
            {{ user.id }} - {{ user.name }}
            <ion-button
            slot="end"
            fill="clear"
            color="danger"
            @click="handleDeleteUser(user.id)"
            >
            <ion-icon :icon="trash" />
            </ion-button>
        </ion-item>
        </ion-list>
    </template>
  
    <script lang="ts">
    import { defineComponent, ref } from 'vue';
    import { IonList, IonItem, IonCheckbox, IonButton, IonIcon, IonLabel } from '@ionic/vue';
    import { User } from '@/models/User';
    import { trash } from 'ionicons/icons';
  
  
    export default defineComponent({
        name: 'UserList',
        props: {
            users: Array as () => User[],
            onUpdateUser: Function,
            onDeleteUser: Function
        },
        components: { IonList, IonItem, IonCheckbox, IonButton, IonIcon, IonLabel },
        setup(props) {
            const ionActiveEl = ref();
            const handleCheckboxChange = (user: User) => {
                // Create a new user object with the updated active value
                const updatedUser = { ...user, active: user.active === 1 ? 0 : 1 };
                if (typeof props.onUpdateUser === 'function') {
                    props.onUpdateUser(updatedUser);
                }
            };
            const handleDeleteUser = (userId: number) => {
                if (typeof props.onDeleteUser === 'function') {
                    props.onDeleteUser(userId);
                }
            }
            return {
                handleCheckboxChange, handleDeleteUser, trash
            };
        },
    });
    </script>
      ```

    !!! Caution Between the </ion-checkbox> <ion-button tags
    You must read `Double left Curly Bracket` user.id `Double Right Curly Bracket` - `Double left Curly Bracket` user.name `Double Right Curly Bracket`


### Modify the Home Page

 The `Home` page will be modify to include:

    - a company logo
    - a welcome text
    - a menu to access application pages

 This will be achieved by creating new Vue components that will be integrated in the `Home` page and for the menu component in the `App.tx` file

 - Create an `AppLogo` Vue component

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
            max-height: 200px;
            display: flex;
            justify-content: center;
        }
        </style>
        ```

    Obviously use your own logo and store it in the `public/assets` folder.

 - Create an `AppIntroText` Vue component

    - In your Editor create a `AppIntroText.vue` file under `src/components`folder and copy the following code

        ```ts
        <template>
            <div class="AppIntroText">
                <h3>
                Welcome to the Ionic7/Vue SQLite Database CRUD App Example Tutorial
                </h3>
                <h4>using @capacitor-community/sqlite</h4>
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
            max-height: 200px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            text-align: center;
            margin-top: 5%;
        }
        </style>
    ```

    Fill free to put whatever text you would like to see

 - Create an `AppMenuButton` Vue component

    - In your Editor create a `AppMenuButton.vue` file under `src/components`folder and copy the following code

        ```ts
        <template>
        <IonMenuButton slot="end" />
        </template>
        
        <script lang="ts">
        import { defineComponent } from 'vue';
        import { IonMenuButton} from '@ionic/vue';
        export default defineComponent({
        name: 'AppMenuButton',
        components: {IonMenuButton}
        });
        </script>
        ```

 - Then the `views/HomePage.vue` file can be modified as below to include all these components:

    ```ts
    <template>
    <ion-page>
        <ion-header :translucent="true">
        <ion-toolbar>
            <app-menu-button></app-menu-button>
            <ion-title>Home</ion-title>
        </ion-toolbar>
        </ion-header>
        <ion-content :fullscreen="true">
        <ion-header collapse="condense">
            <ion-toolbar>
            <ion-title size="large">Home</ion-title>
            </ion-toolbar>
        </ion-header>
        <div id="container">
            <app-logo></app-logo>
            <app-intro-text></app-intro-text>
        </div>
        </ion-content>
    </ion-page>
    </template>

    <script lang="ts">
    import { defineComponent } from 'vue';
    import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar } from '@ionic/vue';
    import AppLogo from '@/components/AppLogo.vue'
    import AppIntroText from '@/components/AppIntroText.vue'
    import AppMenuButton from '@/components/AppMenuButton.vue'

    export default defineComponent({
        name: 'HomePage',
        components: { IonPage, IonHeader, IonToolbar, IonTitle,
            IonContent, AppLogo, AppIntroText, AppMenuButton },
        setup () {

        } 
    });
    </script>

    <style scoped>
    #container {
        text-align: center;  
        position: relative;
        top: 10%;
        left: 0;
        right: 0;
        display: flex;
        flex-direction: column;
    }
    </style>
    ``` 

### Add a Menu to the App 

 - For easy maintenance, the menu is implemented as a Vue component.

   - In your Editor create a `AppMenu.vue` file under `src/components`folder and copy the following code

    ```ts
    <template>
        <ion-menu class="AppMenu" side="end" content-id="main-content">
        <ion-header>
            <ion-toolbar>
            <ion-title>Menu Content</ion-title>
            </ion-toolbar>
        </ion-header>
        <ion-content>
            <ion-list>
            <ion-item @click="closeMenu">
                <ion-button size="default" router-link="/users" expand="full">Managing Users</ion-button>
            </ion-item>
            <!-- ... other menu items -->
            </ion-list>
        </ion-content>
        </ion-menu>
    </template>
  
    <script lang="ts">
    import { IonMenu, IonHeader, IonToolbar, IonTitle, IonContent, IonList,
            IonItem, IonButton } from '@ionic/vue';
    import { defineComponent } from 'vue';
    import { useIonRouter } from '@ionic/vue';

    export default defineComponent({
        name: 'AppMenu',
        components: { IonMenu, IonHeader, IonToolbar, IonTitle, IonContent,
                    IonList, IonItem, IonButton },
        setup() {
            const router = useIonRouter();
    
            const closeMenu = () => {
                const menu = document.querySelector('ion-menu');
                menu!.close();
            };
            return {closeMenu, router}
        },
    });
    </script>
    <style scoped>
        .AppMenu  ion-button {
            --background: transparent;
            --color: initial;
            font-size: 18px;
        }
    </style>
     ```

 - Then modify the `App.vue` file
    - by adding 

        ```ts
        import AppMenu from './components/AppMenu/AppMenu';
        ```

    - replace `<ion-router-outlet />` with

        ```html
        <app-menu />
        <ion-router-outlet id="main-content"/>
        ```

### Update Config and Package.json Scripts

 - For Web application only the appID of the `capacitor.config.ts` must be amended with `YOUR_APP_ID`

 - For the `package .json` file 
    - add the following script below `"build": "tsc && vite build",`,

        ```json
        "build:web": "npm run copy:sql:wasm && vue-tsc && vite build",
        "ionic:serve:before": "npm run copy:sql:wasm",
        "copy:sql:wasm": "copyfiles -u 3 node_modules/sql.js/dist/sql-wasm.wasm public/assets",
        ```

    - modify the `"dev": "vite",` with

        ```json
        "dev": "npm run copy:sql:wasm && vite",
        ```

 - For the `vite.config.ts`file to reduce the size of the chunks after minification add the following lines after the `plugins` block

    ```ts
    build: {
        rollupOptions: {
            output:{
                manualChunks(id) {
                    if (id.includes('node_modules')) {
                        return id.toString().split('node_modules/')[1].split('/')[0].toString();
                    }
                }
            }
        }
    },
    ```

### Run the Web SQLite App

 - To run the Web app use the below command

    ```bash
    npm run dev
    ``` 
 - In your favorite Browser enter

    ```
    http://localhost:5173/
    ```

 - This will bring you to the `Home` page


    <div align="center"><br><img src="/images/Ionic7-Vue-SQLite-HomePage.png" width="250" /></div><br>

 - <p>To open the menu, click on the <span style="font-size: 24px;"><ion-icon name="menu"></ion-icon></span> icon in the top left corner.</p>

   <div align="center"><br><img src="/images/Ionic7-Vue-SQLite-MenuOpen.png" width="250" /></div><br>

 - <p> In the <strong><code>Menu Content</code></strong> click on <strong><code>MANAGING USERS</code></strong></p>
 
   <div align="center"><br><img src="/images/Ionic7-Vue-SQLite-Managing-Users.png" width="250" /></div><br>

On the screen-copy, ones has entered already some users.
 - To input new users
    - type a new user on the input field "Rose Miller"  under Name
    - press on the `"Add User"` button will add the user.

 - To update the `active`value for a user, press on the check icon on the left it will toggle the value from 1 to 0 or from 0 to 1.

 - To delete a user press on the `trash` icon.

 - To save the database to store leave the `UsersPage` by going back `Home`.

The database location can be see in the application tab of the development tools.

<div align="center"><br><img src="/images/Ionic7-Vue-SQLite-Database-Location.png" width="80%" /></div><br>


### Part 1 Conclusion

We have completed the Part 1 - Web application of the Ionic 7 SQLite Database CRUD App Example Tutorial using Vue and @capacitor-community/sqlite.

We learned how to implement the '@capacitor-community/sqlite' plugin in the Vue Framework using standalone components on a Web platform.

We learned to use `app.config.globalProperties` to ensure that the services are available to all components within the app.

We learned how to create an application menu.

We learned how to used some basic methods of the '@capacitor-community/sqlite' plugin to create a CRUD application from scratched and store persistent SQLite data into the Web IndexedDB browser database.

Enjoy your development from there.


## Part 2 - Native - Table of Contents

---
 - [Install Native Required Packages](#install-native-required-packages)
 - [Update Config file](#update-config-file)
 - [Update Package.json Scripts](#update-packagejson-scripts)
 - [Run the iOS App](#run-the-ios-app)
 - [Run the Android App](#run-the-android-app)
 - [Run the Electron App](#run-the-electron-app)
 - [Part 2 Conclusion](#part-1-conclusion)

---

### Install Native Required Packages

 - In order to build Android, iOS, or Electron applications on specific devices, it is necessary to install the necessary capacitor packages first. So run the following commands:

    ```bash
    npm install --save @capacitor/android
    npm install --save @capacitor/ios
    npm install --save @capacitor-community/electron
    ```
 
 - Integrate the various platforms into the application.

    ```bash
    npm run build
    npx cap add android
    npx cap add ios
    npx cap add @capacitor-community/electron
    ```

### Update Config file

The `capacitor.config.ts` file specifies parameters that the plugin needs, such as database location, database encryption, and the use of biometric authentication for access security. Such settings may be platform specific. 

 - Please modify the `capacitor.config.ts` file as below :

    ```ts
    import { CapacitorConfig } from '@capacitor/cli';

    const config: CapacitorConfig = {
    appId: 'YOUR_APP_ID',
    appName: 'YOUR_APP_NAME',
    webDir: 'dist',
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

### Update Package.json Scripts

 - First install a new package to remove the sql.js wasm file to reduce the native package size

    ```bash
    npm install --save-dev rimraf
    ```

 - Open the `package.json` file and replace the scripts section with:

    ```json
    "scripts": {
        "dev": "npm run copy:sql:wasm && vite",
        "build:web": "npm run copy:sql:wasm && npm run build",
        "build:native": "npm run remove:sql:wasm && npm run build",
        "build": "vue-tsc && vite build",
        "preview": "vite preview",
        "ionic:serve:before": "npm run copy:sql:wasm",
        "copy:sql:wasm": "copyfiles -u 3 node_modules/sql.js/dist/sql-wasm.wasm public/assets",
        "remove:sql:wasm": "rimraf public/assets/sql-wasm.wasm",
        "ios:start": "npm run remove:sql:wasm && npm run build:native && npx cap sync && npx cap copy && npx cap open ios",
        "android:start": "npm run remove:sql:wasm && npm run build:native && npx cap sync && npx cap copy && npx cap open android",
        "electron:install": "cd electron && npm install && cd ..",
        "electron:prepare": "npm run remove:sql:wasm && npm run build && npx cap sync @capacitor-community/electron && npx cap copy @capacitor-community/electron",
        "electron:start": "npm run electron:prepare && cd electron && npm run electron:start",
        "clean:vite:cache": "vite clean",
        "test:e2e": "cypress run",
        "test:unit": "vitest",
        "lint": "eslint"
    },
    ```

### Run the iOS App

 - Run the following command:

    ```bash
    npm run ios:start
    ```

 - In Xcode wait for indexed file to complete, clean the project and run the app. You will get the following screen for the Home Page

    <div align="center"><br><img src="/images/Part2-iOS-Ionic7-Vue-SQLite-HomePage.png" width="250" /></div><br>

 - <p>To open the menu, click on the <span style="font-size: 24px;"><ion-icon name="menu"></ion-icon></span> icon in the top left corner.</p>

   <div align="center"><br><img src="/images/Part2-iOS-Ionic7-Vue-SQLite-MenuOpen.png" width="250" /></div><br>

 - <p> In the <strong><code>Menu Content</code></strong> click on <strong><code>MANAGING USERS</code></strong></p>
 
   <div id="banner" style="overflow: hidden; display: flex; justify-content:space-around;">
    <div style="max-width: 40%;"><img src="/images/Part2-iOS-Ionic7-Vue-SQLite-Managing-Users-1.png"/></div>
    <div style="max-width: 38.1035%;"><img src="/images/Part2-iOS-Ionic7-Vue-SQLite-Managing-Users-2.png"/></div>
   </div>

On the screen-copies, ones has entered already some users.
 - To input new users
    - type a new user on the input field "Rose Miller"  under Name
    - press on the `"Add User"` button will add the user.

 - To update the `active`value for a user, press on the check icon on the left it will toggle the value from 1 to 0 or from 0 to 1.

 - To delete a user press on the `trash` icon.

 - To leave the `UsersPage` by going back `Home`.


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
 
 - Screenshot of the screen for the Home page:
 
 <div align="center"><br><img src="/images/Part2-Android-Ionic7-Vue-SQLite-HomePage.png" width="50%" /></div><br>

 - <p>To open the menu, click on the <span style="font-size: 24px;"><ion-icon name="menu"></ion-icon></span> icon in the top left corner.</p>

   <div align="center"><br><img src="/images/Part2-Android-Ionic7-Vue-SQLite-MenuOpen.png" width="250" /></div><br>

 - <p> In the <strong><code>Menu Content</code></strong> click on <strong><code>MANAGING USERS</code></strong></p>
 
   <div id="banner" style="overflow: hidden; display: flex; justify-content:space-around;">
    <div style="max-width: 40%;"><img src="/images/Part2-Android-Ionic7-Vue-SQLite-Managing-Users-1.png"/></div>
    <div style="max-width: 39.81%;"><img src="/images/Part2-Android-Ionic7-Vue-SQLite-Managing-Users-2.png"/></div>
   </div>

On the screen-copies, ones has entered already some users.
 - To input new users
    - type a new user on the input field "Rose Miller"  under Name
    - press on the `"Add User"` button will add the user.

 - To update the `active`value for a user, press on the check icon on the left it will toggle the value from 1 to 0 or from 0 to 1.

 - To delete a user press on the `trash` icon.

 - To leave the `UsersPage` by going back `Home`.


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
        "@capacitor-community/electron": "^4.1.2",
        "@capacitor-community/sqlite": "5.2.3",
        "better-sqlite3-multiple-ciphers": "^8.4.0",
        "chokidar": "~3.5.3",
        "crypto": "^1.0.1",
        "crypto-js": "^4.1.1",
        "electron-is-dev": "~2.0.0",
        "electron-json-storage": "^4.6.0",
        "electron-serve": "~1.1.0",
        "electron-unhandled": "~4.0.1",
        "electron-updater": "~5.0.1",
        "electron-window-state": "~5.0.3",
        "jszip": "^3.10.1",
        "node-fetch": "2.6.7"
    },
    "devDependencies": {
        "@electron/rebuild": "^3.3.0",
        "@types/better-sqlite3": "^7.6.4",
        "@types/crypto-js": "^4.1.1",
        "@types/electron-json-storage": "^4.5.0",
        "electron": "^25.2.0",
        "electron-builder": "^24.6.3",
        "typescript": "~4.3.5"
    },

    ```

 - In your Editor delete the `node_modules` folder and `package-lock.json` file.

 - Then run the following commands:

    ```bash
    npm install
    cd ..
    npm run electron:start
    ```

 - Screenshot of the screen for the Home page:
 
 <div align="center"><br><img src="/images/Part2-Electron-Ionic7-Vue-SQLite-HomePage.png" width="50%" /></div><br>

 - <p>To open the menu, click on the <span style="font-size: 24px;"><ion-icon name="menu"></ion-icon></span> icon in the top left corner.</p>

   <div align="center"><br><img src="/images/Part2-Electron-Ionic7-Vue-SQLite-MenuOpen.png" width="250" /></div><br>

 - <p> In the <strong><code>Menu Content</code></strong> click on <strong><code>MANAGING USERS</code></strong></p>
 
   <div id="banner" style="overflow: hidden; display: flex; justify-content:space-around;">
    <div style="max-width: 40%;"><img src="/images/Part2-Electron-Ionic7-Vue-SQLite-Managing-Users-1.png"/></div>
    <div style="max-width: 39.84%;"><img src="/images/Part2-Electron-Ionic7-Vue-SQLite-Managing-Users-2.png"/></div>
   </div>

On the screen-copies, ones has entered already some users.
 - To input new users
    - type a new user on the input field "Rose Miller"  under Name
    - press on the `"Add User"` button will add the user.

 - To update the `active`value for a user, press on the check icon on the left it will toggle the value from 1 to 0 or from 0 to 1.

 - To delete a user press on the `trash` icon.

 - To leave the `UsersPage` by going back `Home`.


### Part 2 Conclusion

We have completed the Part 2 - Native and Electron applications of the Ionic 7 SQLite Database CRUD App Example Tutorial using Vue framework and @capacitor-community/sqlite.

We have learned to adapt the `capacitor.config.ts`file and the modify the scripts: section of the `package.json`file for native apps

We learned how to build and run the application for each native platforms.

Enjoy your development from there.

