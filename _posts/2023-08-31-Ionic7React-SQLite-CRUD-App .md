
# Ionic 7 SQLite Database CRUD App Example Tutorial using React and @capacitor-community/sqlite
---

*last updated on September 3, 2023 by Quéau Jean Pierre*

In that tutorial we will learned how to create a Ionic7/React basic CRUD application and implement the @capacitor-community/sqlite plugin to store the data in a SQLite database.

The first part of the tutorial will concentrate on how to create that application and run it on a Web browser where the data will be stored on an Indexed database using sql.js and localForage modules.
Go to [Part 1 - Web - Table of Contents](#part-1---web---table-of-contents)

Thanks to the Ionic Team and their hard work to bring CAPACITOR 5, the second part will concentrate on native platforms (iOS and Android) and also on Electron platform.
Go to [Part 2 - Native - Table of Contents](#part-2---native---table-of-contents)


## Part 1 - Web - Table of Contents

---
 - [Install New Ionic Application](#install-new-ionic-application)
 - [Install Required Packages](#install-required-packages)
 - [Create a React Cli Config file](#create-a-react-cli-config-file)
 - [Create the SQLite Database](#create-the-sqlite-database)
 - [Create CRUD and SQLite Services](#create-crud-and-sqlite-services)
 - [React Plugin Implementation](#react-plugin-implementation)
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
    sudo npm install -g npm install g generate-react-cli
    ```

 - Create a new blank Ionic React app

    ```bash
    ionic start ionic7-react-sqlite-app blank --type=react --capacitor
    ```

 - Go inside the project folder

    ```bash
    cd ./ionic7-react-sqlite-app
    ```

### Install Required Packages

 - run the below commands

    ```bash
    npm install --save @capacitor-community/sqlite
    npm install --save @capacitor/toast
    npm install --save @ionic/pwa-elements
    npm install --save-dev copyfiles
    ```

### Create a React Cli Config file

 - In your editor, create a `generate-react-cli.json` file and includes the following code:

    ```json
    {
    "usesTypeScript": true,
    "usesStyledComponents": false,
    "testLibrary": "None",
    "component": {
        "default": {
        "path": "src/components",
        "withStyle": true,
        "withTest": false,
        "withStory": false,
        "withLazy": false
        }
    },
    "usesCssModule": false,
    "cssPreprocessor": "css"
    }
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

    - `services/SQLiteService.ts`
    - `services/StorageService.ts`

 - Open the `services/sqlite.service.ts` file and replace with the following code.

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

### React Plugin Implementation

 For Web applications, the plugin uses the Stencil component `jeep-sqlite` which implies some react files modification

 - Open the `main.tsx` file and replace the code with the following

    ```ts
    import React from 'react';
    import { createRoot } from 'react-dom/client';
    import App from './App';

    import { Capacitor } from '@capacitor/core';
    import { JeepSqlite } from 'jeep-sqlite/dist/components/jeep-sqlite';
    import { defineCustomElements as pwaElements} from '@ionic/pwa-elements/loader';

    pwaElements(window);
    customElements.define('jeep-sqlite', JeepSqlite);
    const platform = Capacitor.getPlatform();

    const rootRender = () => {
        const container = document.getElementById('root');
        const root = createRoot(container!);
        root.render(
            <React.StrictMode>
            <App />
            </React.StrictMode>
        );
    }
    if (platform !== "web") {
        rootRender();
    } else {
        window.addEventListener('DOMContentLoaded', async () => {
            const jeepEl = document.createElement("jeep-sqlite");
            document.body.appendChild(jeepEl);
            customElements.whenDefined('jeep-sqlite').then(() => {
                rootRender();
            })
            .catch ((err) => {
                console.log(`Error: ${err}`);
                throw new Error(`Error: ${err}`)
            });
        });
    }
    ``` 

 - To initialize the application, an `AppInitializer` React component is used and has to be created

    ```bash
    npx generate-react-cli component AppInitializer
    ```

 - In your editor, open the file `components/AppInitializer/AppInitializer.tsx` and replace the code with:

    ```ts
    import { FC, useEffect, useContext, useRef } from 'react';
    import { Toast } from '@capacitor/toast';
    import './AppInitializer.css';
    import InitializeAppService  from '../../services/initializeAppService';
    import { SqliteServiceContext, StorageServiceContext } from '../../App';

    interface AppInitializerProps {
        children : any
    }

    const AppInitializer: FC<AppInitializerProps> = ({ children }) => {
        const ref = useRef(false);
        const sqliteService = useContext(SqliteServiceContext);
        const storageService = useContext(StorageServiceContext);
        const initializeAppService = new InitializeAppService(sqliteService,
                                                                storageService);
        useEffect(()=> {
            const initApp = async ():Promise <void> => {
                try {
                    const appInit = await initializeAppService.initializeApp();
                    return;
                } catch(error: any) {
                    const msg = error.message ? error.message : error;
                    Toast.show({
                    text: `${msg}`,
                    duration: 'long'
                    });           
                }  
            };
            if(ref.current === false) {
                initApp();
                ref.current = true;
            }

        }, []);

        return <>{children}</>;
    };

    export default AppInitializer;
    ```

 The `AppInitializer` component is defined using the FC type. It takes children as a prop and uses the useEffect hook to run the initialization logic when the component mounts. The {children} element is used to render the wrapped components passed as children in fact here the application. 
 To make the code more modular and easier to maintain, the `AppInitializer` component then simply calls the `initializeApp` method from the `initializeService` to perform the necessary initialization tasks.

 - Create  in your Editor the `services/initializeService.ts` file and replace the code with

    ```ts
    import {platform} from '../App';
    import {ISQLiteService } from '../services/sqliteService'; 
    import {IStorageService } from '../services/storageService'; 

    export interface IInitializeAppService {
        initializeApp(): Promise<boolean>
    };

    class InitializeAppService implements IInitializeAppService  {
        appInit = false;
        sqliteServ!: ISQLiteService;
        storageServ!: IStorageService;

        constructor(sqliteService: ISQLiteService, storageService: IStorageService) {
            this.sqliteServ = sqliteService;
            this.storageServ = storageService;
        }
        async initializeApp() : Promise<boolean> {
            if(!this.appInit) {
                try {
                    if (platform === 'web') {
                        const jeepSQlEL = document.querySelector("jeep-sqlite")
                        await this.sqliteServ.initWebStore();
                    }
                    // Initialize the myuserdb database
                    await this.storageServ.initializeDatabase();
                    if( platform === 'web') {
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

 - Finally open the `App.tsx` file  and replace the code with

    ```ts
    import React from 'react';
    import { Redirect, Route } from 'react-router-dom';
    import { IonApp, IonRouterOutlet, setupIonicReact } from '@ionic/react';
    import { IonReactRouter } from '@ionic/react-router';
    import Home from './pages/Home';
    import { Capacitor } from '@capacitor/core';
    import SqliteService from './services/sqliteService';
    import DbVersionService from './services/dbVersionService';
    import StorageService  from './services/storageService';
    import AppInitializer from './components/AppInitializer/AppInitializer';

    import UsersPage from './pages/UsersPage/UsersPage';
    import AppMenu from './components/AppMenu/AppMenu';

    /* Core CSS required for Ionic components to work properly */
    import '@ionic/react/css/core.css';

    /* Basic CSS for apps built with Ionic */
    import '@ionic/react/css/normalize.css';
    import '@ionic/react/css/structure.css';
    import '@ionic/react/css/typography.css';

    /* Optional CSS utils that can be commented out */
    import '@ionic/react/css/padding.css';
    import '@ionic/react/css/float-elements.css';
    import '@ionic/react/css/text-alignment.css';
    import '@ionic/react/css/text-transformation.css';
    import '@ionic/react/css/flex-utils.css';
    import '@ionic/react/css/display.css';

    /* Theme variables */
    import './theme/variables.css';

    export const platform = Capacitor.getPlatform();

    // Singleton Services
    export const SqliteServiceContext = React.createContext(SqliteService);
    export const DbVersionServiceContext = React.createContext(DbVersionService);
    export const StorageServiceContext = React.createContext(new StorageService(SqliteService,DbVersionService));


    setupIonicReact();

    const App: React.FC = () => {
        return (
            <SqliteServiceContext.Provider value={SqliteService}>
                <DbVersionServiceContext.Provider value={DbVersionService}>
                    <StorageServiceContext.Provider value={new StorageService(SqliteService,DbVersionService)}>
                        <AppInitializer>
                            <IonApp>
                            <IonReactRouter>
                                <AppMenu />
                                <IonRouterOutlet id="main-content">
                                <Route exact path="/home">
                                    <Home />
                                </Route>
                                <Route exact path="/">
                                    <Redirect to="/home" />
                                </Route>
                                <Route path="/users" component={UsersPage} />
                                </IonRouterOutlet>
                            </IonReactRouter>
                            </IonApp>
                        </AppInitializer>
                    </StorageServiceContext.Provider>
                </DbVersionServiceContext.Provider>
            </SqliteServiceContext.Provider>
        )
    };

    export default App;
    ``` 
 As the goal is to ensure that the services (`SqliteService`, `DbVersionService` and `StorageService`) are unique and properly shared and available throughout the components, a Context Provider was created for Each Service.

 By wrapping your App component structure with the context providers and the AppInitializer component, you ensure that the services are available to all components within your app. The dependency injection mechanism will ensure that you'll have access to your shared services in both the HomePage and the UsersPage, as well as any other components that you include in your routes or other parts of your app in the future.

### Users Page Implementation

 As you can see above, a route `/users` to a `UsersPage` has been added.
 Let's create it.

 - In your Editor, create a new folder `UsersPage` under pages folder and in that new folder create two files `UsersPage.css` and `UsersPage.tsx`.

 - Then open the `UsersPage.tsx` file and add the following code:

    ```ts
    import { useState, useEffect, useRef, useContext } from 'react';
    import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonCard,
            IonButtons, IonBackButton, useIonViewWillEnter, useIonViewWillLeave
        } from '@ionic/react';
    import './UsersPage.css';
    import UserForm from '../../components/UserForm/UserForm';
    import UserList from '../../components/UserList/UserList';
    import { User } from '../../models/User';
    import { useQuerySQLite } from '../../hooks/UseQuerySQLite';
    import { SQLiteDBConnection } from "@capacitor-community/sqlite";
    import { platform } from '../../App';
    import { SqliteServiceContext, StorageServiceContext } from '../../App';
    import { Toast } from '@capacitor/toast';

    const UsersPage: React.FC = () => {
        const ref = useRef(false);
        const dbNameRef = useRef('');
        const isInitComplete = useRef(false);
        const [users, setUsers] = useState<User[]>([]);
        const [db, setDb] = useState<SQLiteDBConnection | null>(null);
        const sqliteServ = useContext(SqliteServiceContext);
        const storageServ = useContext(StorageServiceContext);

        const openDatabase = () => {
            try {
                const dbUsersName = storageServ.getDatabaseName();
                dbNameRef.current = dbUsersName;
                const version = storageServ.getDatabaseVersion();

                sqliteServ.openDatabase(dbUsersName, version, false).then((database) => {
                    setDb(database);
                    ref.current = true;
                });
            } catch (error) {
                const msg = `Error open database:: ${error}`;
                console.error(msg);
                Toast.show({
                    text: `${msg}`,
                    duration: 'long'
                });           
            }
        }

        const handleAddUser = async (newUser: User) => {
            if (db) {
                // Send the newUser to the addUser storage service method
                const isConn = await sqliteServ.isConnection(dbNameRef.current,false);
                const lastId = await storageServ.addUser(newUser);
                newUser.id = lastId;

                // Update the users state to include the newly added user
                setUsers(prevUsers => [...prevUsers, newUser]);

            }
        };

        const handleUpdateUser = async (updUser: User) => {
            if (db) {
                const isConn = await sqliteServ.isConnection(dbNameRef.current,false);
                await storageServ.updateUserById(updUser.id.toString(),updUser.active);
                // Update the users state with the modified user
                setUsers(prevUsers =>
                    prevUsers.map(user =>
                    user.id === updUser.id ? { ...user, active: updUser.active } : user
                    )
                );
            }
        };

        const handleDeleteUser = async (userId: number) => {
            if (db) {
                const isConn = await sqliteServ.isConnection(dbNameRef.current,false);
                await storageServ.deleteUserById(userId.toString());
                // Update the users state by filtering out the deleted user
                setUsers(prevUsers => prevUsers.filter(user => user.id !== userId));
            }
        };

        useIonViewWillEnter( () => {
            const initSubscription = storageServ.isInitCompleted.subscribe  ((value) => {
                isInitComplete.current = value;
                if(isInitComplete.current === true) {
                    const dbUsersName = storageServ.getDatabaseName();
                    if(ref.current === false) {
                        if(platform === "web") {
                            customElements.whenDefined('jeep-sqlite').then(() => {
                                openDatabase();
                            })
                            .catch ((error) => {
                                const msg = `Error open database:: ${error}`;
                                console.log(`msg`);
                                Toast.show({
                                    text: `${msg}`,
                                    duration: 'long'
                                });           
                            });
                        } else {
                            openDatabase();
                        }
                    }
                }
            });

            return () => {
                initSubscription.unsubscribe();
            };
        }, [storageServ]);

        useIonViewWillLeave(  () => {

            sqliteServ.closeDatabase(dbNameRef.current,false).then(() => {
                ref.current = false;  
            })
            .catch((error) => {
                const msg = `Error close database:: ${error}`;
                console.error(msg);
                Toast.show({
                text: `${msg}`,
                duration: 'long'
                });           
            });
        });
        useEffect(() => {
            // Fetch users from the database using useQuerySQLite hook

            if (isInitComplete.current === true && db) {
                const stmt = 'SELECT * FROM users';
                const values: any[] = [];
                const fetchData =  useQuerySQLite(db, stmt, values);
                fetchData()
                .then((fetchedUserData) => {
                    setUsers(fetchedUserData);
                })
                .catch((error) => {
                    const msg = `Error fetching user data: ${error}`;
                    console.error(msg);
                    Toast.show({
                        text: `${msg}`,
                        duration: 'long'
                    });           
                });
            }
        }, [db]);


        return (
            <IonPage>
            <IonHeader>
                <IonToolbar>
                <IonTitle>Managing Users</IonTitle>
                <IonButtons slot="start">
                    <IonBackButton text="home" defaultHref="/home"></IonBackButton>
                </IonButtons>
                </IonToolbar>
            </IonHeader>
            <IonContent>
                {ref.current && (
                <div>
                    <IonCard>
                        <h1>Add New User</h1>
                        <UserForm onAddUser={handleAddUser} />
                    </IonCard>
                    <IonCard>
                        <h2>Current Users</h2>
                        <UserList users={users} onUpdateUser={handleUpdateUser} 
                        onDeleteUser={handleDeleteUser}/>
                    </IonCard>
                </div>
                )}
            </IonContent>
            </IonPage>
        );
    };

    export default UsersPage;
    ```



### Users Component Implementation

 In that tutorial we use a simple UI interface for CRUD operations through the use of a `UserForm` and a `UserList` React component.
 
 - Run the below command

    ```bash
    npx generate-react-cli component UserForm
    npx generate-react-cli component UserList
    ```
 
 - Open the `components/UserForm/UserForm.tsx` file and replace the code with

    ```ts
    import { FC, useState, useEffect, useRef } from 'react';
    import { User } from '../../models/User';
    import { IonInput, IonButton } from '@ionic/react';

    import './UserForm.css';

    interface UserFormProps {
        onAddUser: (user: User) => void;
    }

    const UserForm: FC<UserFormProps> = ({ onAddUser }) => {
        const isFormValid = useRef(false);
        const [name, setName] = useState('');
        const ionNameEl = useRef<HTMLIonInputElement>(null);

        /* for database Version 2
        const [email, setEmail] = useState('');
        const ionEmailEl = useRef<HTMLIonInputElement>(null);
        */

        const handleNameInput = (ev: Event) => {
            const value = (ev.target as HTMLIonInputElement).value as string;

            // Removes non alphanumeric characters
            const filteredValue = value.replace(/[^a-zA-Z0-9-' ']+/g, '');

            // Update both the state variable and
            //  the component to keep them in sync.
            setName(filteredValue);

            const inputCmp = ionNameEl.current;
            if (inputCmp !== null) {
                inputCmp.value = filteredValue;
            }

        };
        /* Version 2
        const handleEmailInput = (ev: Event) => {
            const value = (ev.target as HTMLIonInputElement).value as string;

            // Update both the state variable and
            // the component to keep them in sync.

            setEmail(value);

            const inputCmp = ionNameEl.current;
            if (inputCmp !== null) {
                inputCmp.value = value;
            }

        };
        */  

        const handleSubmit = () => {
            if (name /*&& email */) {
                const newUser: User = {
                    id: Date.now(), // do not care about the id value (generated by sqlite)
                    name,
                    active: 1,
                    /*email, //version 2 */
                };

                onAddUser(newUser);

                setName('');
                /*setEmail('');  // version 2 */
                isFormValid.current = false;
            }
        };
        const hasTwoWords = (input: string): boolean => {
            // Split the input string by spaces
            const words = input.trim().split(' ');

            // Check if there are exactly two non-empty words
            const isTwo = words.length === 2 && words.every(word => word !== '');
            return isTwo;
        }

        /* Version 2
        const isValidEmail = (email: string): boolean => {
            if(email.length === 0){
                return false;
            }
            const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{1,}$/;
            const test = emailRegex.test(email);
            return test;
        }
        */
        useEffect(()=> {
            (email) // version 2*/ ;
        }, [name /*, email // version 2*/]);

        return (
            <div className="UserForm">
                <IonInput
                    ref={ionNameEl}
                    type="text"
                    value={name}
                    onIonInput={handleNameInput}
                    placeholder="Rose Miller"
                ></IonInput>
                {/* version 2
                <IonInput
                    ref={ionEmailEl}
                    type="email"
                    value={email}
                    onIonInput={handleEmailInput}
                    placeholder="Email"
                ></IonInput>
                */}
                <IonButton expand="full" onClick={handleSubmit} disabled={!isFormValid.current}>
                    Add User
                </IonButton>
            </div>
        )
    };

    export default UserForm;
    ``` 
 - Open the `components/UserList/UserList.tsx` file and replace the code with

    ```ts
    import { FC } from 'react';
    import './UserList.css';
    import { IonList, IonItem, IonCheckbox, IonButton, IonIcon, IonLabel } from '@ionic/react';
    import { User } from '../../models/User';
    import { trash } from 'ionicons/icons'; // Import the trash icon

    interface UserListProps {
        users: User[],
        onUpdateUser: (user: User) => void;
        onDeleteUser: (userId: number) => void; 
    }

    const UserList: FC<UserListProps> = ({users, onUpdateUser, onDeleteUser}) => {
        const handleCheckboxChange = (user: User) => {
            // Create a new user object with the updated active value
            const updatedUser = { ...user, active: user.active === 1 ? 0 : 1 };
            onUpdateUser(updatedUser);
        };

        return (
            <IonList className="UserList">
                { users.map((user: User) => (
                    <IonItem key={user.id}>
                        <IonCheckbox
                            slot="start"
                            aria-label=""
                            checked={user.active === 1}
                            onIonChange={() => handleCheckboxChange(user)}>
                        </IonCheckbox>
                        <IonLabel>
                        {user.id} - {user.name}
                        </IonLabel>
                        <IonButton
                            slot="end"
                            fill="clear"
                            color="danger"
                            onClick={() => onDeleteUser(user.id)}>
                            <IonIcon icon={trash} />
                        </IonButton>
                    </IonItem>
                ))}
            </IonList>
        )
    };

    export default UserList;
    ```

### Modify the Home Page

 The `Home` page will be modify to include:

    - a company logo
    - a welcome text
    - a menu to access application pages

 This will be achieved by creating new React components that will be integrated in the `Home` page and for the menu component in the `App.tx` file

 - Create an `AppLogo` React component

    ```bash
    npx generate-react-cli component AppLogo 
    ```
    - In your Editor replace the code of `AppLogo/AppLogo.tsx` with 

        ```ts
        import React, { FC } from 'react';
        import './AppLogo.css';

        interface AppLogoProps {}

        const AppLogo: FC<AppLogoProps> = () => (
        <div className="AppLogo">
            <img src="https://avatars3.githubusercontent.com/u/16580653?v=4" width="128" height="128" />
        </div>
        );

        export default AppLogo;
        ```
    Obviously use your own logo

    - In your Editor replace the code of `AppLogo/AppLogo.css` with 

        ```css
        .AppLogo {
            width: 100%;
            max-height: 200px;
            display: flex;
            justify-content: center;
        }
        ```
 - Create an `AppIntroText` React component

    ```bash
    npx generate-react-cli component AppIntroText 
    ```
    - In your Editor replace the code of `AppIntroText/AppIntroText.tsx` with 

        ```ts
        import React, { FC } from 'react';
        import './AppIntroText.css';

        interface AppIntroTextProps {}

        const AppIntroText: FC<AppIntroTextProps> = () => (
        <div className="AppIntroText">
            <h3>
            Welcome to the Ionic7/React SQLite Database CRUD App Example Tutorial
            </h3>
            <h4>using @capacitor-community/sqlite</h4>
        </div>
        );

        export default AppIntroText;
        ```
    Fill free to put whatever text you would like to see

    - In your Editor replace the code of `AppIntroText/AppIntroText.css` with 

        ```css
        .AppIntroText {
            margin: 100px;
            display: flex;
            flex-direction: column;
            text-align: center;
        }
        ```
 - Create an `AppMenuButton` React component

    ```bash
    npx generate-react-cli component AppMenuButton 
    ```
    - In your Editor replace the code of `AppMenuButton/AppMenuButton.tsx` with 

        ```ts
        import { FC } from 'react';
        import './AppMenuButton.css';
        import { IonMenuButton } from '@ionic/react';

        interface AppMenuButtonProps {}

        const AppMenuButton: FC<AppMenuButtonProps> = () => {
        return (
            <IonMenuButton slot="end" />
        )
        };

        export default AppMenuButton;
        ```

 - Then the `pages/Home.tsx` file can be modified as below to include all these components:

    ```ts
    import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar } from '@ionic/react';
    import AppLogo from '../components/AppLogo/AppLogo';
    import AppMenuButton from '../components/AppMenuButton/AppMenuButton';
    import './Home.css';
    import AppIntroText from '../components/AppIntroText/AppIntroText';

    const Home: React.FC = () => {
        return (
            <IonPage>
                <IonHeader>
                    <IonToolbar>
                        <AppMenuButton />
                        <IonTitle>Home</IonTitle>
                    </IonToolbar>
                </IonHeader>
                <IonContent>
                    <AppLogo />
                    <AppIntroText />
                </IonContent>
            </IonPage>
        );
    };

    export default Home;
    ``` 

### Add a Menu to the App 

 - For easy maintenance, the menu is implemented as a React component.

    ```bash
    npx generate-react-cli component AppMenu
    ```
 - In your Editor, modify the `AppMenu/AppMenu.tsx` file

    ```ts
    import React, { FC } from 'react';
    import './AppMenu.css';
    import { IonMenu, IonHeader, IonToolbar, IonTitle, IonContent,
            IonList, IonItem, IonButton} from '@ionic/react';

    interface AppMenuProps {}

    const AppMenu: FC<AppMenuProps> = () => {
    const closeMenu = () => {
        const menu = document.querySelector('ion-menu');
        menu!.close();
    };

    return (
        <IonMenu className="AppMenu" side="end" contentId="main-content">
            <IonHeader>
                <IonToolbar>
                <IonTitle>Menu Content</IonTitle>
                </IonToolbar>
            </IonHeader>
            <IonContent>
                <IonList>
                <IonItem onClick={closeMenu}>
                    <IonButton size="default" routerLink="/users" expand="full">Managing Users</IonButton>
                </IonItem>
                {/* ... other menu items */}
                </IonList>
            </IonContent>
        </IonMenu>
    )
    };
    export default AppMenu;
    ```

 - In your Editor, modify the `AppMenu/AppMenu.css` file

    ```css
    .AppMenu ion-button {
        --background: transparent;
        --color: initial;
        font-size: 18px;
    }
    ```

 - Then modify the `App.tsx` file
    - by adding 

        ```ts
        import AppMenu from './components/AppMenu/AppMenu';
        ```

    - replace `<IonRouterOutlet>` with

        ```html
        <AppMenu />
        <IonRouterOutlet id="main-content">
        ```

### Update Config and Package.json Scripts

 - For Web application only the appID of the `capacitor.config.ts` must be amended with `YOUR_APP_ID`

 - For the `package .json` file 
    - add the following script below `"build": "tsc && vite build",`,

        ```json
        "copy:sql:wasm": "copyfiles -u 3 node_modules/sql.js/dist/sql-wasm.wasm public/assets",
        ```

    - modify the `"dev": "vite",` with

        ```json
        "dev": "npm run copy:sql:wasm && vite",
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


    <div align="center"><br><img src="/images/Ionic7-React-SQLite-HomePage.png" width="250" /></div><br>

 - <p>To open the menu, click on the <span style="font-size: 24px;"><ion-icon name="menu"></ion-icon></span> icon in the top left corner.</p>

   <div align="center"><br><img src="/images/Ionic7-React-SQLite-MenuOpen.png" width="250" /></div><br>

 - <p> In the <strong><code>Menu Content</code></strong> click on <strong><code>MANAGING USERS</code></strong></p>
 
   <div align="center"><br><img src="/images/Ionic7-React-SQLite-Managing-Users.png" width="250" /></div><br>

On the screen-copy, ones has entered already some users.
 - To input new users
    - type a new user on the input field "Rose Miller"  under Name
    - press on the `"Add User"` button will add the user.

 - To update the `active`value for a user, press on the check icon on the left it will toggle the value from 1 to 0 or from 0 to 1.

 - To delete a user press on the `trash` icon.

 - To save the database to store leave the `UsersPage` by going back `Home`.

The database location can be see in the application tab of the development tools.

<div align="center"><br><img src="/images/Ionic7-React-SQLite-Database-Location.png" width="80%" /></div><br>


### Part 1 Conclusion

We have completed the Part 1 - Web application of the Ionic 7 SQLite Database CRUD App Example Tutorial using React and @capacitor-community/sqlite.

We learned how to implement the '@capacitor-community/sqlite' plugin in the React Framework using standalone components on a Web platform.

We learned to use Context providers to ensure that the services are available to all components within your app.

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
        "build": "vite build",
        "preview": "vite preview",
        "copy:sql:wasm": "copyfiles -u 3 node_modules/sql.js/dist/sql-wasm.wasm public/assets",
        "remove:sql:wasm": "rimraf public/assets/sql-wasm.wasm",
        "ios:start": "npm run remove:sql:wasm && npm run build:native && npx cap sync && npx cap copy && npx cap open ios",
        "android:start": "npm run remove:sql:wasm && npm run build:native && npx cap sync && npx cap copy && npx cap open android",
        "electron:install": "cd electron && npm install && cd ..",
        "electron:prepare": "npm run remove:sql:wasm && npm run build && npx cap sync @capacitor-community/electron && npx cap copy @capacitor-community/electron",
        "electron:start": "npm run electron:prepare && cd electron && npm run electron:start",
        "test.e2e": "cypress run",
        "test.unit": "vitest",
        "lint": "eslint"

    },
    ```

### Run the iOS App

 - Run the following command:

    ```bash
    npm run ios:start
    ```

 - In Xcode wait for indexed file to complete, clean the project and run the app. You will get the following screen for the Home Page

    <div align="center"><br><img src="/images/Part2-iOS-Ionic7-React-SQLite-HomePage.png" width="250" /></div><br>

 - <p>To open the menu, click on the <span style="font-size: 24px;"><ion-icon name="menu"></ion-icon></span> icon in the top left corner.</p>

   <div align="center"><br><img src="/images/Part2-iOS-Ionic7-React-SQLite-MenuOpen.png" width="250" /></div><br>

 - <p> In the <strong><code>Menu Content</code></strong> click on <strong><code>MANAGING USERS</code></strong></p>
 
   <div id="banner" style="overflow: hidden; display: flex; justify-content:space-around;">
    <div style="max-width: 40%;"><img src="/images/Part2-iOS-Ionic7-React-SQLite-Managing-Users-1.png"/></div>
    <div style="max-width: 38,1035%;"><img src="/images/Part2-iOS-Ionic7-React-SQLite-Managing-Users-2.png"/></div>
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
 
 <div align="center"><br><img src="/images/Part2-Android-Ionic7-React-SQLite-HomePage.png" width="50%" /></div><br>

 - <p>To open the menu, click on the <span style="font-size: 24px;"><ion-icon name="menu"></ion-icon></span> icon in the top left corner.</p>

   <div align="center"><br><img src="/images/Part2-Android-Ionic7-React-SQLite-MenuOpen.png" width="250" /></div><br>

 - <p> In the <strong><code>Menu Content</code></strong> click on <strong><code>MANAGING USERS</code></strong></p>
 
   <div id="banner" style="overflow: hidden; display: flex; justify-content:space-around;">
    <div style="max-width: 40%;"><img src="/images/Part2-Android-Ionic7-React-SQLite-Managing-Users-1.png"/></div>
    <div style="max-width: 40%%;"><img src="/images/Part2-Android-Ionic7-React-SQLite-Managing-Users-2.png"/></div>
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
        "@capacitor-community/sqlite": "^5.0.7",
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
        "@types/better-sqlite3": "^7.6.4",
        "@types/crypto-js": "^4.1.1",
        "@types/electron-json-storage": "^4.5.0",
        "electron": "^25.2.0",
        "electron-builder": "^24.6.3",
        "electron-rebuild": "^3.2.9",
        "rimraf": "^5.0.1",
        "typescript": "~4.3.5"
    },

    ```

 - Then run the following commands:

    ```bash
    npm install
    cd ..
    npm run electron:start
    ```

 - Screenshot of the screen

 <div align="center"><br><img src="/images/Part2-Electron-Ionic7-React-SQLite-Managing-Users.png" width="50%" /></div><br>

### Part 2 Conclusion

We have completed the Part 2 - Native and Electron applications of the Ionic 7 SQLite Database CRUD App Example Tutorial using React framework and @capacitor-community/sqlite.

We have learned to adapt the `capacitor.config.ts`file and the modify the scripts: section of the `package.json`file for native apps

We learned how to build and run the application for each native platforms.

Enjoy your development from there.

